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

## 24. 区别universal reference与rvalue reference

* `T&&`这样的形式，在大多数情况下是右值引用，但某些情况下可以是左值引用。实际上，`T&&`还可以引用const、volatile类型的对象。由于它的这种前所未有的弹性，本书作者将它称为`universal reference`。

* 满足下面两个两件的对象的类型将是`universal reference`：

  1. 变量的类型必须通过类型推导确定（因此对象可能被推导为右值引用或左值引用）；
  2. 声明形式必须是`T&&`，不能有const、volatile等修饰符。

* 有两种场景会满足上面的两个条件，导致出现`universal reference`：

```c++
template<typename T>
void func(T&& param); // param是universal reference

auto&& var2 = var1; // var2是universal reference
```

* 以下场景看似满足上面的条件，但实际上没有满足：

```c++
template<typename T>
void f(std::vector<T>&& param); // 声明形式不是T&&，因此param是右值引用

template<typename T>
void func(const T&& param); // 因为有const,param一定是右值引用

template<typename T>
class vector {
public:
  void push_back(T&& x); // 这里只有模板类的推导，x是不需要推导的，因此x是右值引用
};
```

## 25. 右值引用使用std::move，universal引用使用std::forward

* 函数接收右值引用，就是为了要move它。同理universal引用也是，就是为了forward它。

* 显然，函数内部在最后一次使用右值引用时才用std::move，否则值是错乱的。同理，要在最后一次使用universal引用时才用std::forward。

* 如果函数以传值方式返回，也适用于上面的规则，返回是属于对该值的最后一次使用。但注意下面一种情况：

   * 如果是传值返回函数内局部变量，这时不要用move或forward，因为这会破坏编译器RVO。编译器对传值返回局部变量会自动优化为move操作，不会有任何拷贝发生。

## 26. 避免重载universal引用

* 假设某个模板函数接受universal引用，如果对这个函数进行重载（参数类型不同的重载，而不是参数个数不同的重载），经常会发现实际调用与预期不符。因为universal引用非常贪婪，会拦截很多引数是引用的情况。

* 尤其是将universal引用被用在构造函数时（构造函数是模板函数），上述问题会表现的更严重，它的子类对基类的构造调用也会被拦截。

* 鉴于以上情况，尽量避免universal引用。

## 27. 让自己熟悉重载universal引用的替代方案

* 本章主要是针对上一条提到的universal引用的副作用提出各种替代方案：

  1. 前三个方案就是告诉你不要用universal引用。

  2. 中间一个方案是：不要重载，而是将原函数的实现改为调用具体的实现函数，针对不同的类型调用不同的实现函数。

  3. 最后一个方法比较杀手锏，使用enable_if模板，让所有不符合预期的传入模板的类型都不能编译通过。

* C++11的[type traits](http://en.cppreference.com/w/cpp/header/type_traits)头文件有非常多的全新内容。

## 28. 认识reference collapsing

* 很多类型推导会推导出`引用的引用`。这时只会生成一层引用，不会真的出现`引用的引用`，这叫做“reference collapsing”。

* 其他内容不是很有用，就是列举了集中会出现`reference collapsing`的情况。

## 29. 假设移动操作不存在、成本没有较低也不会被使用

* 江湖上流传着这样的传说：C++11的STL做了大幅修正，支持了移动，之前的c++98时代的代码使用支持c++11的代码重新编译后性能都会有提高。

* 上面这种说法是有夸大的成分。很多时候编译器可能并不会自动生成移动操作（例如第17条说的场景），c++11并不会让代码性能更好。

* 有以下原因会导致c++11的移动特性不会使程序性能更好：

  * 对象没有搬移操作。移动操作被编译为复制操作。

  * 移动操作没有更快。例如std::array的复制和移动操作的时间复杂度都是O(n)。

  * 无法使用移动。很多场景要求移动操作不能抛出异常，但移动操作函数没有声明为noexcept

  * 来源对象是lvalue。除了少数例外（第25条），只有rvalue才能被移动。

* 我们编写模板时，无法确定类型是否支持移动操作，是否移动操作的成本较低，也不知道是否会用到移动操作。所以我们要假设移动操作不存在，假设成本没有较低，假设不会用到移动操作。
