---
title:  "python packaging"
categories: 
  - Python
---

pip19开始支持pep517，pipenv最新版也支持。可以用pipenv安装poetry项目(https://github.com/python-poetry/poetry/issues/321)。pep517未明确editable install， 不能用-e参数；poetry自身支持

## poetry vs pipenv
- poetry支持build, 可以用poetry写library, 其他poetry项目可直接引用（pipenv项目引用时有问题）;pipenv 只解决 application(concrete) dependency management
