---
title:  "postgresql explain"
categories: 
  - postgres
---

## explain基本结构
- 树状节点，第一行和以->开头的行是一个节点，每个node代表一个operation，node下的几行文字是这个节点/操作的额外说明(比如`Seq Scan`的`FILTER`)。上层的节点需要下层节点的输出，下层节点执行完了下层节点才能执行。
- 数字含义：(cost=1.14..107.61 ROWS=808 width=138) (actual TIME=42.495..54.854 ROWS=71 loops=1) 第一个小括号是explain，第二个小括号是explain analyze。第一个是预测的、相对的单位，width是一行的bytes大小；第二个是实际的ms,loops是operation执行的次数。

## query plan中不同类型的node/operation

### Seq Scan
sequencial scan，full scan，电子书中的long query

这种如果有limit是可以返回partial结果的，不一定全扫完。像存储过程中循环return一个值再sleep，这种加limit仍然要等存储过程全部执行完毕

### INDEX Scan

### INDEX Scan Backward

### INDEX ONLY Scan

### FUNCTION Scan
针对的是`function that returns recordset `，如 `SELECT * FROM generate_Series(1,10) i;`

### Sort

### Limit
够数后，这个节点可以`stops sub-operation afterwards, but in some cases (pl/PgSQL functions for example), the sub-operation is already finished when it returned first row`。function中的return的结果是临时存在cache中的，结束了cache中的结果再一起返回。

### HashAggregate
对应group by，每个key对应一个bucket，sub-operation通常是一个scan。下边的scan全部完成后，每个bucket输出一个结果
> HashAggregate does something like this: for every row it gets, it finds GROUP BY “key" (in this case relkind). Then, in hash (associative array, dictionary), puts given row into bucket designated by given key.
>
> After all rows have been processed, it scans the hash, and returns single row per each key value, when necessary – doing appropriate calculations (sum, min, avg, and so on).
>
> It is important to understand that HashAggregate has to scan all rows before it can return even single row.

### Hash JOIN / Hash
跟nested loop类似，永远有两个sub-operation，其中一个必是Hash。先执行Hash（通常包含子scan），再循环另一个sub-op(通常是scan)针对每个元素去找对应的bucket
> First Hash Join calls “Hash", which in turns calls something else (Seq Scan on pg_namespace in our case). Then, Hash makes a memory (or disk, depending on size) hash/associative-array/dictionary with rows from the source, hashed using whatever is used to join the data (in our case, it's OID column in pg_namespace)
>
> Then, Hash Join runs the second suboperation (Seq Scan on pg_class in our case), and for each row from it, it does:
>   1. check if join key (pg_class.relnamespace in our case) is in hash returned by Hash operation
>   2. if it is not – given row from suboperation is ignored (will not be returned)
>   3. if it exists – Hash Join fetches rows from hash, and based on row from one side, and all rows from hash, it generates output rows


### Nested Loop
第一个op是外层循环，第二个op是内层循环。
> 1. Nested Loop runs one side of join, once. Let's name it “A".
> 2. For every row in “A", it runs second operation (let's name it “B")
> 3. if “B" didn't return any rows – data from “A" is ignored
> 4. if “B" did return rows, for every row it returned, new row is returned by Nested Loop, based on current row from A, and current row from B

### MERGE JOIN / MERGE
排序后再join
> 1.if join column on right side is the same as join column on left side:
>   - return new joined row, based on current rows on the right and left sides
>   - get next row from right side (or, if there are no more rows, on left side)
>   - go to step 1
> 
> 2.if join column on right side is “smaller" than join column on left side:
>   - get next row from right side (if there are no more rows, finish processing)
>   - go to step 1
> 
> 3.if join column on right side is “larger" than join column on left side:
>   - get next row from left side (if there are no more rows, finish processing)
>   - go to step 1

### Hash Join / Nested Loop / Merge Join modifiers
> We can have LEFT/RIGHT/FULL outer joins. And there are so called anti-joins.
> In case of left/right joins, the operation names get changed to:
> - Hash Left Join
> - Hash Right Join
> - Merge Left Join
> - Merge Right Join
> - Nested Loop Left Join
> There is no Nested Loop Right Join, because Nested Loop always starts with left side as basis to looping. So join that uses RIGHT JOIN, that would use Nested Loop, will get internally transformed to LEFT JOIN so that Nested Loop can work.
>
> In all those cases the logic is simple – we have two sides of join – left and right. And when side is mentioned in join, then join will return new row even if the other side doesn't have matching rows.
> 
> There is also a version called Full Join, with operation names:
> - Hash Full Join
> - Merge Full Join
> In which case join generates new output row regardless of whether data on either side is missing (as long as the data is there for one side)
>
> There are also so called Anti Joins. Their operation names look like:
> - Hash Anti Join
> - Merge Anti Join
> - Nested Loop Anti Join
> In these cases Join emits row only if the right side doesn't find any row. This is useful when you're doing things like “WHERE not exists ()" or “left join … where right_table.column is null".

### Materialize
> Materialize gets data from underlying operation and stores it in memory (or partially in memory) so that it can be used faster, or with additional features that underlying operation doesn't provide.
比如Nested Loop的right part，materialize后第2个op就只需要执行一次了，后续的loop直接读内存。

### Unique
removes duplicate data（distinct），But in newer Pgs this query will usually be done using HashAggregate
这个操作需要数据是有序的
> This makes it really cool (when possible to use) because it doesn't use virtually any memory. It just checks if value in previous row is the same as in current, and if yes – discards it. That's all.
> 
> So, we can force usage of it, by pre-sorting data:

### Append
对应union / union all 
> UNION removes duplicate rows – which is, in this case, done using HashAggregate operation.
> 
> HashAggregate  
>   -> Append (.....)

### Result
> This operation is used when your query selects some constant value (or values)

### Values Scan
> returning simple, entered in query, data, but this time – it can be whole recordset, based on VALUES() functionality
>
> SELECT * FROM ( VALUES (1, 'hubert'), (2, 'depesz'), (3, 'lubaczewski') ) AS t (a,b);

### GroupAggregate
> This is similar to previously described HashAggregate.
>
> The difference is that for GroupAggregate to work data has to be sorted using whatever column(s) you used for your GROUP BY clause.
> 
> Just like Unique – GroupAggregate uses very little memory, but forces ordering of data.

### HashSetOp
> This operation is used by INTERSECT/EXCEPT operations (with optional “ALL" modifier).
>
>It works by running sub-operation of Append for a pair of sub-queries, and then, based on result and optional ALL modifier, it figures which rows should be returned. I haven't digged in the source code so I can't tell you exactly how it works, but given the name and the operation, it looks like a simple counter-based solution.

### CTE Scan
> This is similar to previously mentioned Materialized operation. It runs a part of a query, and stores output so that it can be used by other part (or parts) of the query.
> The very important thing is that CTEs are ran just as specified. So they can be used to circumvent some not-so-good optimizations that planner normally can do.
 
### InitPlan
> This plan happens whenever there is a part of your query that can (or have to) be calculated before anything else, and it doesn't depend on anything in the rest of your query.
>
> Pg correctly sees that the subselect column does not depend on any data from pg_class table, so it can be run just once, and doesn't have to redo the length-calculation for every row.
>
> There is one important thing, though – numbering of init plans within single query is “global", and not “per operation".

### SubPlan
> SubPlans are a bit similar to NestedLoop. In this way that these can be called many times.
>
> SubPlan is called to calculate data from a subquery, that actually does depends on current row.

## 参考
- [Explaining the unexplainable](https://www.depesz.com/tag/unexplainable/) 系列
- 电子书 [PostgreSQL Query Optimization](https://www.amazon.com/PostgreSQL-Query-Optimization-Ultimate-Efficient/dp/1484268849)
