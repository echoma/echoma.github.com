---
layout: post
title:  "《effective python》笔记 第3章 类与继承"
date:   2017-02-21 14:03:21 +0800
---

> 善用类和继承，可以写出易于维护的代码

## 22. 尽量使用异常来表示特殊情况，而不要返回None

* 不要使用多层嵌套字典来表示对象，也不要使用过长的元组，否则代码会变得复杂难懂。应当使用类来对数据对象进行抽象表达。

* 使用`collections`模块中的`namedtuple`可以非常轻松的定义出精简且不可变的数据类型：

```python
import collections
Grade = collections.namedtuple('Grade', ('score', 'weight'))
```

* `namedtuple`也有它的局限性：它不能指定各参数的默认值，对于有可选属性的数据来说不太方便。另外，`namedtuple的`的属性仍然是可以通过下表和遍历访问的，可能会导致不太符合设计者意图的的使用方式。

* 一些复杂的数据可以拆解为多个辅助类，代码会更清晰、更易扩展。