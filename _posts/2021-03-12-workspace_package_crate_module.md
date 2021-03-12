---
layout: post
title:  "workspace, package, crate, module in rust"
categories: 
  - Rust
---

# Module
## Privacy
```
mod outer {
	mod a {
		pub mod aa {
		}
	}
	fn b(){}
}
```
a是b的sibling，b可以使用a中的pub item（functions, methods, structs, enums, modules, and constants）。a不需要加pub（a加pub含义是让module outer的同辈可以访问）；aa是b的child module，aa需要加pub才能让b访问。
  
能访问到module的名字，就能访问到module下的pub items（b能访问module名a，可以把module a想像成fn a，同一级当然可以访问；b能访问a后，就能访问a的pub item：module aa）

(the book 7.3)
### Structs
Structs have an extra level of visibility with their fields. The visibility defaults to private, and can be overridden with the pub modifier. This visibility only matters when a struct is accessed from outside the module where it is defined, and has the goal of hiding information (encapsulation).  
(rust by example 10.2)
