---
layout: post
title:  "《effective modern c++》笔记 第5章 右值引用、移动语义和完美转发"
date:   2017-05-05 13:17:21 +0800
---

* 目录
{:toc}

# 第五章

* 移动语义的应用场景：

  1. 让编译器用低成本的移动取代高成本的赋值操作。
  2. 类似`std::unique_ptr`、`std::thread`这类组件是无法复制的，只能移动。

* 完美转发的应用场景：能够建立一种模板，该模板接受任意个数的因数，并将引数全部转发给目标函数。具体的场景在26条中给出描述。

* 右值引用(rvalue reference)、移动语义(move)、完美转发(perfect forwarding)中有些让初学者感觉模糊不清的地方。本书将解释相关功能的来龙去脉，列举使用惯例，让读者了解各种意外表现背后的原因。

> 需要特别注意：所有的参数都是左值(lvalue)，即使参数类型是右值引用(rvalue reference)。

## 23. 认识std::move和std::forward

* 移动操作的源一定不能是const的。对const对象进行移动操作，编译器会尝试调用复制操作。

```c++
// 调用someFunc()函数，实际上输出的是“as lvalue”

void process(const std::string& lval) // 处理左值
{
  cout << "as lvalue" << endl;
}

void process(std::string&& rval) // 处理右值
{
  cout << "as rvalue" << endl;
}

void someFunc()
{
  const string str = "some string";
  process(std::move(str));
}
```

* `std::move()`不会产生任何执行码，**该函数只做了转型，并没有做移动操作**。该函数返回了一个参数的右值。该函数的命名原因是：可以获得用作移动来源的对象。

* `std::move()`并不保证转型后的右值一定能够移动。该函数唯一能保证的，就是它会返回右值(rvalue)。

* `std::forward()`跟move()类似，只是它的转型是有条件的。它只在引数以rvalue初始化时才会转型为右值。

* `std::forward()`的使用场景，我们以下面这个例子做解释。

  1. 首先main()中对`someFunc()`的两次调用是有不同的需求的，两次调用的代码注释里有解释。

  2. 对于`someFunc()模板`内部的实现，使用move机制、forward机制、不使用任何机制的表现是完全不同的，代码注释有详细解释。

```c++
void process(const std::string& lval) // 处理左值
{
  cout << " in process, as lvalue, c_str\'s addr=" << (void*)lval.c_str() << endl;
}

void process(std::string&& rval) // 处理右值
{
  cout << " in process, as rvalue, c_str\'s addr=" << (void*)rval.c_str() << endl;
  string new_dest(move(rval)); // 进行移动操作
  cout << " in process, after move, dstr\'s addr=" << (void*)new_dest.c_str() << endl;
}

template<typename T>
void someFunc(T&& str)
{
  // process(move(str)); // 使用move机制，main()中的两次调用最终都会调用process()的处理右值的版本
  // process(str); // 不用任何机制，main()中的两次调用都会最终调用process()的处理左值的版本（str的类型是string&，推导规则在第1节有详细说明）
  process(forward<T>(str)); // 使用forward机制，会自动根据main()中的调用形式，转发给不同的process()版本
}

int main()
{
  string str = "some string";
  cout << "1st call, out process,  c_str\'s addr=" << (void*)str.c_str() << endl;
  someFunc(str); // 不希望str被移动，因为我们后面还要使用str进行其他调用
  cout << "after 1st call, str=" << str << endl;
  cout << "2nd call, out process,  c_str\'s addr=" << (void*)str.c_str() << endl;
  someFunc(move(str)); // 可以移动，因为我们不会对字符串做其他的使用了
  cout << "after 2nd call, str=" << str << endl; // 会输出空字符串，因为被移动了
  return 0;
}
```

* 总结：move操作表示无条件转换为rvalue，forward表示只在rvalue时才转为rvalue。move通常会造成移动操作，forward只是对lvalue/rvalue属性的完美转发。
