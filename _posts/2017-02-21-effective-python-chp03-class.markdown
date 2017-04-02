---
layout: post
title:  "《effective python》笔记 第3章 类与继承"
date:   2017-02-21 14:03:21 +0800
---

* 目录
{:toc}

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

* 上面例子中的函数是无状态的，如果想要函数保存状态或携带某些属性，就只能使用类了。python的类通过实现简单的`__call__`接口就能够像函数那样被调用：

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

## 25. 用super初始化父类

* 初始化父类最原始的方法是在__init__函数中调用父类的__init__，这种方法在多重继承、菱形继承中可能会出现以下问题：
  1. 调用多个父类的__init__的顺序跟类声明中父类的顺序不同
  2. 菱形继承中难以确定各个成员的初始化顺序

* 从python2.2开始引入了`super`，使用它可以按照确定的顺序来初始化父类（深度优先，从左至右）。`super`是标准的初始化方案，应当总是使用它来初始化父类。

* python3的super语法比pyhon2更加简洁、清晰。

## 26. 只在使用Mix-in组件制作工具类时进行多重继承

* `Min-in`组件是一种小型类，它实现了某种接口，其他类可以通过继承它来快速具备该接口。

* 多重继承会带来很多迷惑、二义性的问题，应当尽量避免使用多重继承。

## 27. 多用public属性，少用private属性

* python的类成员的可见度只有两种：`public`和`private`，以双下划线`__`开头的成员就是private的。但python没有严格的保证private字段的私密性，通过`obj._ClassName__private_field`是可以直接访问的。原因有二：

  1. `我们都是成年人`，python程序员门认为开放比封闭好。
  2. private仅用来减少无意间访问内部属性带来的意外。

* 程序员通过命名来识别字段的可见度，`PEP8`中规定单下划线`_`开头的字段应当被开发者视为protected。建议多用protected属性，并在文档中说明这些字段的用法，不要视图用private属性来进行限制。

* 只在一种情况考虑使用private：为了避免不受控制的子类跟父类之间的成员命名冲突。

## 28. 继承collections.abc以实现自定义容器

* 通常，要实现自定义容易可能需要实现非常多成员方法，比如`__getitem__(self, index)`：下标访问方法，可以像`bar[0]`这样访问；比如`__len__(self)`：使内置的len可以工作，像`len(bar)`这样访问。不同的容器类型有各种不同的需要实现的方法，程序员很难记忆。

* 我们可以使用`collections.abc`，它为每种容器类型都设定了必须要实现的方法，如果我们漏实现了某个方法，该模块会指出这个错误：

```python
from collections.abc import Sequence

class BadType(Sequence):
    pass

foo = BadType()

>>>
TypeError: Cant't instancetiate abstract class BadType with
    abstract methods __getitem__, __len__
```

* 如果我们提供了必须的方法，该模块会自动帮我们实现其他方法。比如对于`Sequence`，如果我们实现了`__getitem__`和`__len__`，该模块会帮我们实现`__count__`和`__index__`方法。