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


## 参考
- [Explaining the unexplainable](https://www.depesz.com/2013/04/16/explaining-the-unexplainable/) 系列
- 电子书 [PostgreSQL Query Optimization](https://www.amazon.com/PostgreSQL-Query-Optimization-Ultimate-Efficient/dp/1484268849)
