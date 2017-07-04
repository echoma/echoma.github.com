---
layout: post
title:  "《effective modern c++》笔记 第6章 lambda表达式"
date:   2017-06-26 13:17:21 +0800
---

* 目录
{:toc}

# 第六章

* lamda虽然没有给语言带来新的表达能力，但是它使函数对象的创建变得极其方便，从而改变了C++程序设计的游戏规则。

* lamda相关术语很容易混淆，因此我们要先说明以下几个术语：

  - lambda表达式(lambda expression)，是个表达式，是一段原始代码。
  - 闭包(closure)，是lambda表达式建立的运行时对象(runtime object)。根据捕获模式(capture mode)的不同，closure会拥有数据的副本或引用。
  - 闭包类(closure class)，是实例化closure的类型，编译器会为每个lambda表达式生成一个独一无二的闭包类。

* 看如下代码：

```c++
int x;
auto c1 = [x](int y) { return x*y > 55; }; // `=`右侧的代码是lambda表达式，c1是一个闭包(closure)实例。
auto c2 = c1; // c2是c1的副本。闭包类是可以拥有多个闭包实例的。
```

## 31. 避免使用预设捕获模式

* 预设捕获模式是指`[&]按传引用自动捕获`和`[=]按传值自动捕获`这两种，这两种模式都有各自的问题。

* `按传引用自动捕获`的问题是：被引用的变量可能是局部变量，closure被执行时局部变量可能已经被释放，造成访问错误内存地址的问题。

* `[=]按传值自动捕获`的问题是：会给人造成两种种错觉：

  - 认为捕获的变量都是传值，因此不会发生访问错误内存地址的问题。但实际上`[=]`捕获的可能是个this指针，this指针是可能被外部代码释放的（事例代码中在一个类成员函数中包含的lambda表达式里访问了类成员，编译器其实捕获的是this指针，而且不是智能指针，就是原始的指针）。

  - 认为使用这种捕获方式后，lambda表达式就自给自足了，不需要依赖外部变量了。但实际上，传值捕获只会对局部变量起作用，外部的静态变量仍然会被依赖。

## 32. 使用初始化捕获讲对象移进closure

* 有时我们既不想用引用捕获也不想用传值捕获，因为可能某个对象只能使用`移动`操作。C++11是没有提供这方面支持的，但是C++14是支持的，这个特性叫做`初始化捕获(init capture)`。

* 通过初始化捕获，我们能够指定：

  - lambda产生的closure类中`成员变量的名称`。
  - 初始化成员变量的`表达式`。

```c++
auto func = [pwd = std::move(pw)]{ return pwd->isValidated(); };
```

* C++11中可以使用`std::bind`来模拟移动捕获。

```c++
auto func = std::bind(
  [](const std::vector<dobule& data) {/*一些使用data变量的代码*/},
  std::move(data)
);
```

* 上面代码中`参数data是const引用`的解释：lambda表达式建立的`closure class`内部的operator()成员函数是const的。但是bind object内部的构建的data并不是const的。因此我们要把参数data主动声明为const。

## 33. 对auto&&参数使用decltype，再做std::forward

* c++14中的`generic lambda`可以让参数使用auto。其实它的实现很简单，就是让lambda的closure class中operator()成员函数成为模板。

* 本章就是教给读者写一个可以`进行完美转发`且支持`不定参数个数(variadic)`的lambda。思路较长，不做记录了，直接给结论：

```c++
// 这里的func和normalize都是一些用户函数。

// 这是最普通的版本
auto f = [](auto x) { return func(normalize(x)); }

// 这是支持完美转发、不定参数的版本
auto f = [](auto&&... param)
{
  return
  func(normalize(std::forward<decltype(params)>(params)...));
}
```

## 34. 尽量使用lambda取代std::bind

* 本章主要针对熟悉std::bind的读者。本人不熟悉，只看了前面导言和后边结论。内容部分跳过。

* 本章要说明的结论：
  - c++11的lambda几乎在所有情况总是比std::bind更为合适，到了c++14后，lambda的优势就更加明显了。
  - lambda比std::bind更具可读性、更有表达力，有时也更有效率。
  - 仅在c++11中，std::bind在一个非常复杂的使用场景下有用。（这个场景的名字没看懂，不抄写了。。。）