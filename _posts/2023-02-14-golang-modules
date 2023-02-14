---
title:  "golang modules"
categories: 
  - Golang
---
go最开始完全是基于package的，module是后加的，所以它们关联没那么紧密，module不过是给项目下各个package提供了一个公共的namespace，用于引用。

package的名称也不一定跟文件夹保持一致，`import`语句使用的是文件夹路径，但package被import后，使用这个package需要用`package`语句定义的名称。比如

- go.mod (github.com/sify21/xx)
- packageA
  - a.go (package xa)
- cmd
  - xxx
    - a.go (可以是package main，而且可以`import github.com/sify21/xx/packageA`，使用时需要`xa.functionName`)
