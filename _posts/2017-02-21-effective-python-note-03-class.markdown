---
layout: post
title:  "《effective python》笔记 第3章 类与继承"
date:   2017-02-21 14:03:21 +0800
---

> 善用类和继承，可以写出易于维护的代码

## 22. 尽量使用辅助类来维护程序状态，而不要使用字典和元组

* 不要使用多层嵌套字典来表示对象，也不要使用过长的元组，否则代码会变得复杂难懂。应当使用类来对数据对象进行抽象表达。

* 使用`collections`模块中的`namedtuple`可以非常轻松的定义出精简且不可变的数据类型：

```python
import collections
Grade = collections.namedtuple('Grade', ('score', 'weight'))
```

* `namedtuple`也有它的局限性：它不能指定各参数的默认值，对于有可选属性的数据来说不太方便。另外，`namedtuple的`的属性仍然是可以通过下标和遍历访问的，可能会导致不太符合设计者意图的的使用方式。

* 一些复杂的数据可以拆解为多个辅助类，代码会更清晰、更易扩展。

## 23. 简单的接口应该接受函数，而不是类的实例

* python内置的很多api允许调用者传入函数或对象来定制api的行为。例如：

```python
names = ['LiLei', 'HanMeimei', 'Peter']
names.sort(key=lambda x: len(x)) # 按名字长度进行排序
```

* 其他语言由于语言本身的限制，通过定义类对象并传入函数来实现类似功能。但函数在python中是一级对象，可以作为参数传递，这种方式比类对象可以写出更加清晰的代码。

* 上面例子中的函数是无状态的，如果想要函数保存状态，就只能使用类了。python的类通过实现简单的`__call__`接口就能够像函数那样被调用：

```python
class SortBySex(object):
    def __init__(self):
        # 定义了一个女士名单
        self.females = ['HanMeimei']
    def __call__(self, name):
        # 按照性别排序，女士排在最前面
        return 0 if name in self.females else 1

names.sort(key=SortBySex())
```

* 上面的代码会比使用闭包函数更易维护。带状态的闭包函数阅读起来会略微难懂。

## 24. 以@classmethod形式多态去通用的构建对象

* 首先需自行了解`@classmethod`语法。

* 本条技巧主要是将使用`@classmethod`可以达到这种效果：`统一的一个接口函数，生成多种类对象`，可以少些代码。（echo：个人觉得很难被用的技巧，全看开发者自己是否想到这样写）
