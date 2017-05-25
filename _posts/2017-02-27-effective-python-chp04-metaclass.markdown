---
layout: post
title:  "《effective python》笔记 第4章 元类及属性"
date:   2017-02-27 19:34:21 +0800
---

* 目录
{:toc}

> 元类及属性是python非常强大的特性，不小心使用会创建出古怪的行为。使用时，应当遵循最小惊讶原则，只适合用来实现广为人知的编程范式。

## 29. 用纯属性取代get和set方法

* 从其他语言转入python的开发者可能会再累中明确的实现getter和setter方法。但python有特有的`@property`修饰器用来完成这点。对于python语言来说，类属性应当总是从最简单的public属性开始：

```python
class MyCls(object):
    def __init__(self):
        self.voltage = 0
```

* 假如今后向实现特殊行为，可以改用`@perperty`修饰器和`setter方法`来做：

```python
class MyCls(object):
    def __init__(self):
        self._voltage = 0
    @property
    def voltage(self):
        return self._voltage
    @property.setter
    def voltage(self, voltage):
        self._voltage = voltage / 10
```

* 执行效率上，这比自己实现getter和setter函数要高。

* 不应当在`setter方法`里实现奇怪的逻辑，比如在一个属性的setter方法里修改另一个属性的值，再比如在setter方法里实现缓慢的开销很大的操作。这些都是调用者不会想到的。

## 30. 考虑用@property来代替属性重构

* `@property`可以在保持原有调用代码无改动的情况下改变类的逻辑，特别适合重构。

* 当发现自己正在大量使用`@property`时，很可能意味着这个类设计的很差，需要彻底重构。

## 31. 用描述符来改写需要复用的@property方法

* `@property`修饰器的代码是无法复用的。假如某个类有几个属性都需要类似的setter逻辑，那就要为每个属性都编写雷同的property代码，这样的代码不易维护。

* 使用描述符提供的`__get__()`和`__set__()`方法，可以复用代码：

```python
class Grade(object):
    def __init__(self):
        self._value = 0
    def __get__(self, instance, instance_type):
        return self._value
    def __set__(self, instance, value):
        self._value = value

class Exam(object):
    def __init__(self):
        self.math_grade = Grade()
        self.writing_grade = Grade()
        self.science_grade = Grade()

exam = Exam()
exam.math_grade = 40
```

> 注意：上文的这种写法是完全错误的！Grade的__set__函数完全没有被调用。math_grade必须是类成员（而不是对象的成员）才可以。python官方文档的例子也只有类成员的例子。

## 32. 用\_\_getattr\_\_、\_\_getattributer\_\_和\_\_setattr\_\_实现按需生成的属性

* 自行了解这几个概念即可。

* 需要注意，如果在`__getattr__`中访问其他属性，如果该属性又引起新的`__getattr__`调用，就会造成死循环。为了避免这种情况，可以使用`super().__getattr__(name)`来访问属性，这不会引起新的`__getattr__`调用。

## 33. 用元类来验证子类

* 首先自行了解元类的概念。

* 注意，在python2和python3中元类的继承方式是不同的。

```python
// python3
class MyClass(object, metaclass=Meta):
    ...
// python2
class MyClass(object):
    __metaclass__ = Meta
```

* 本条目主要就是说可以在元类的`__init__`方法中对构造参数的正确性做统一的检查。

## 34. 用元类来注册子类

* 本条目主要是说在元类的`__init__`方法中保存子类的名字和类型的映射关系，从而可以实现反射机制。

## 35. 用元类来注解子类

* 本条目主要是说在元类的`__init__`方法中可以对子类的成员进行增删改操作，可以做到让所有子类都表现出某种一致的行为。这可以大量减少代码重复。