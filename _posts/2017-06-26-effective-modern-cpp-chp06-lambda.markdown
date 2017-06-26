---
layout: post
title:  "《effective modern c++》笔记 第6章 lambda表达式"
date:   2017-05-05 13:17:21 +0800
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
