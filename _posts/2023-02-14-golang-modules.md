---
title:  "golang vs rust code structure"
categories: 
  - Golang
  - Rust
---
## Golang代码结构：workspace, module, package

go最开始完全是基于package的，依赖引用也是package之间的引用，写在源码的`import`语句里。

module是后加的，所以它们关联没那么紧密，module不过是给项目下各个package提供了一个公共的namespace。可以在go.mod里指定这个module的具体位置在哪，这样这个module下的package的位置也就确定了。原来没module的时候需要把所有的package都放在`GOROOT`下。

package的名称也不一定跟文件夹保持一致，`import`语句使用的是文件夹路径，但package被import后，使用这个package需要用`package`语句定义的名称。比如

- go.mod (github.com/sify21/xx)
- packageA
  - a.go (package xa)
- cmd
  - xxx
    - a.go (可以是package main，而且可以`import github.com/sify21/xx/packageA`，使用时需要`xa.functionName`)

go1.18新增了workspace, 这个是处理同时在多个module上工作的情况。go.work里的replace语句可以解决多个module的go.mod replace同一个module是冲突的情况，比如这几个module引用依赖module时用的是相对路径，可能相对的路径都不一致，这时就可以在go.work中解决这个冲突。

## Rust代码结构: workspace, package, crate, mod(ule)

依赖引用的单位是crate（准确的说是lib-crate），mod是crate内部进行代码划分的。

crate可以是bin或lib，一个package可以包含最多一个lib-crate和多个bin-crate。

`Cargo.toml`既可以用来声明workspace，又可以用来声明package，`cargo publish`的时候发布的是package对应的lib-crate
