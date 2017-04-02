---
layout: post
title:  "《effective modern c++》笔记 第3章 迈入现代C++"
date:   2017-03-09 13:45:21 +0800
---

* 目录
{:toc}

# 第三章

> C++11/14有大量的重大新特性，在学习这些之前，必须先要学习一些基础的知识，本章将会一一介绍。

## 7. 构造对象时使用()与{}的区别

* 让我们来看下各种变量初始化的三种形式：

  1. `int x(0);` 初始值在小括号内
  2. `int y = 0; ` 初始值在=号右侧
  3. `int z{ 0 }; 或 int z = { 0 }; ` 初始值在大括号内

* 上面三种形式在其他场景的应用情况：

  1. 容器初始化，只能用`{ }`大括号形式：`std::vector<int> v{1,2,3}`。

  2. C++11的非静态成员初始化不能使用小括号：

  ```c++
  class Foo {
  private:
    int x{0}; // OK
    int y = 0; // OK
    int z(0); // 编译错误
  };
  ```

  3. 无法赋值的对象（如std::atomic，第40条会介绍），不能使用=号初始化：

  ```c++
  std::atomic<int> x{0}; // OK
  std::atomic<int> y(0); // OK
  std::atomic<int> z = 0; // 编译错误
  ```

* 通过上面的场景可以发现，`{ }`是应用范围最广的初始化形式，被称为`大括号初始化列表(braced-init-list)`语法，这是c++11引入的`通用(uniform)`初始化形式，适用于所有场景。

* `{ }`形式还有与其他形式与众不同的地方：

  1. 还可以避免`向下转型(narrowing conversion)`：

  ```c++
  int a {0.2}; // 编译错误
  double x = 0.2;
  int b {x}; // 编译错误
  int c (x); // OK
  int d = x; // OK
  ```

  2. C++会把所有能视为声明的东西看做是声明，这有时候有副作用：

  ```c++
  Widget w1(10); // 用10构造对象w1
  Widget w2(); // 本意是像使用默认值构造对象w2，但实际上被视为声明了函数w2
  Widget w3{}; // 用默认值构造对象w3
  ```

* 通过上面的阐述我们发现，使用`{}`形式好处多多。那为什么本节的标题不直接叫做`优先使用{}初始化语法`呢？因为有几个特殊情况：

  1. `{}`会被编译器优先推导为`std::initializer_list`类型，如果我们使用`auto`关键字声明对象，那对象的类型会被推导为`std::initializer_list`类型。这一点我们在第2条有说过，我们还提到了草案N3922。因此，在使用`auto`声明变量时，不建议使用`{}`形式初始化。
  2. 如果某个类，支持`std::initializer_list`参数的构造函数，那在使用`{}`形式初始化该类的对象时，编译器优先使用`std::initializer_list`参数的构造函数。

* echo：关于上面说到的草案N3922，这里做进一步阐述。该草案对类型推导在使用`大括号初始化列表` 时的推到规则做了更改：
  * `auto a = {};`被叫做`拷贝初始化列表(copy-init-list)`，而`auto a{};`叫做`直接初始化列表(direct-init-list)`。
  * 对于`拷贝初始化列表`，`auto`将永远被推导为`std::initializer_list`。
  * 对于`直接初始化列表`，`{}`内只允许有1个元素，`auto`会被推导为该元素的类型。因此，类似`auto a{};`和`auto a{1,2}`此类语法是无法编译通过的。
  * `c++14`已经禁止在返回类型使用`auto`的函数里，使用`{}`形式的返回值进行推导。

## 8. 尽量用nullptr取代0或NULL

* 关于`0`、`NULL`和`nullptr`的基本阐述：
  * `0`的问题是：`0`是整数，而不是指针。c++在该使用指针的地方碰到0时，会勉为其难的解释为空指针。
  * `NULL`的问题是：具体的C++库在实现时，NULL可能会被定义为int、long等整数类型，并不是指针类型。在调用一些重载函数时，会引发一些混乱。
  * 而`nullptr`实际上是`std::nullptr_t`类型，该类型可以自动转换为所有类型的指针。

  ```c++
  void f(int)
  void f(void*)
  void f(bool)

  f(0); // 会调用f(int)，而不是f(void*)
  f(NULL); // 通常会调用f(int)，不会调用f(void*)。有些编译器可能无法编译。
  ```