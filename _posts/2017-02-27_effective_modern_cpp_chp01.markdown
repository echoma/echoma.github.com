---
layout: post
title:  "《effective modern c++》笔记 导读+第1章 类型推导"
date:   2017-02-27 13:06:21 +0800
---

> `auto`和`decltype`的使用正在越来越广泛，使用类型推导写出的代码更能适应变化，更易维护。只要在一个地方修改类型，通过类型推导就可以使改动扩散到程序其他地方。但是，代码不能直观呈现类型推导的结果，书写正确代码的难度也变高了。为了更有效率的书写代码，本章提供了所有C++开发人员需要了解的类型推导知识。

# 全书导读

* `C++版本`：书中对C++98、C++03统称为C++98，谈到C++11时指C++11与C++14标准，谈到C++14则专指C++14。因为98和03标准区别很小，14是11的超集。

* `引数`与`参数`：调用函数是传入的变量叫做`引数`(argument)，而引数初始化函数的`参数`(parameter)。例如：

```c++
void someFunc(Widget w); // w是参数

Widget wid;
someFunc(wid); // wid是引数
```

* `异常安全`：设计良好的函数都具备`异常安全`的特性，表示至少提供基本的异常安全保证(basic guarantee)，即使抛出异常仍能够保证不会发生数据结构损毁，不会泄露资源。如果提供强异常安全保证(strong guarantee)，则发生异常时，程序的状态与调用前相同（类似事务回滚）。
* `闭包`：使用lambda表达式创建的函数对象叫做`闭包`，通常是不需要区分lambda表达式和闭包的，我们会统称lambda。
* `通用引用`：universal reference，作者提出的一种称呼。顾名思义，可以接受任何类型的引用。读本书过程中会时不时的碰到这个概念，但是不需要理解清楚，第24条会做出详细说明。

# 第一章

## 1. 了解模板类型推导

* 在了解推导规则前，先看下模板函数的声明及调用的模型：

```c++
template<typename T>
void f(ParamType param);
f(expr); // 由expr推导出T和ParamType
```

* 推导情况一：ParamType是指针或引用，但不是通用引用。推导规则如下：
  1. 如果expr是引用，则去掉引用部分；
  2. 根绝expr的类型跟ParamType样式作模式比对(pattern match)得到T。

```c++
template<typename T>
void f1(T& param); // ParamType是T&，是个引用，符合本情况。

template<typename T>
void f2(const T& param); // ParamType是const T&，是个引用，符合本情况。

template<typename T>
void f3(T* param); // ParamType是T*，是个引用，符合本情况。

int x = 27;
// expr的类型是int
f1(x); // int比对T&，推导T为int
f2(x); // int比对const T&，推导T为int

const int cx = x;
// expr的类型是const int
f1(cx); // const int比对T&，推导T为const int
f2(cx); // const int比对const T&，推导T为int

const int& rx = x;
// expr的类型是const int&，需要去掉引用部分，变成const int
f1(rx); // const int比对T&，推导T为const int
f2(rx); // const int比对const T&，推导T为int

// expr的类型是int*
f3(&x); // int*比对T*，推导T为int

// expr的类型是const int*
f3(&cx); // const int*比对T*，推导T为const int
```

* 推导情况二：ParamType是通用引用。情况会不太直观，参数的声明类似右指引用`T&&`（第24条会对左值、右值做详细说明）。推导规则如下：
  1. 如果expr是左值，则T和ParamType都会推导为左值的引用；
  2. 如果expr是右值，则使用情况一的规则进行推导。

```c++
template<typename T>
void f(T&& param); // ParamType是通用引用，符合本情况

int x = 27;
// expr的类型是int，且是左值
f(x); // T推导为int&，param的类型也是int&

const int cx = x;
// expr的类型是const int，且是左值
f(cx); // T推导为const int&，param的类型也是const int&

const int& rx = x;
// expr的类型是const int&，且是左值
f(rx); // T推导为const int&，param的类型也是const int&

// expr的类型是int，是右值，使用情况一个规则进行推导
f(27); // int比对T&&，T推导为int，param的类型是int&&
```

* 推导情况三：ParamType不是指针也不是引用（那就一定是传值了）。推导规则如下：
  1. 如果expr是引用，则去掉引用。
  2. 如果是const，则去掉const。（因为传值，所以参数是个拷贝，肯定是可以修改的）
  3. 如果是volatile，则去掉volatile

```c++
template<typename T>
void f(T param); // ParamType就是传值

int x = 27;
// expr的类型是int
f(x); // T推导为int

const int cx = x;
// expr的类型是const int，去掉const，变为int
f(cx); // T推导为int

const int& rx = x;
// expr的类型是const int&，先去掉引用，再去掉const，变为int
f(rx); // T推导为int

const char* ptr = "test";
// expr的类型是const char*，指针本身不是const，不需要去掉const
f(ptr); // T推导为const char*

const char* const ptr2 = "test2";
// expr的类型是const char* const，去掉const，变为const char*
f(ptr2); // T推导为const char*
```

* 引数为数组时的特别推导规则：
  1. 如果模板参数为传值，则数组退化为指针，使用情况三的规则推导；
  2. 如果模板参数为引用，则保持数组类型，使用情况一的规则推导。

```c++
const char name[] = "J. P. Briggs"; //name类型是const char[13]

template<typename T>
void f1(T param); // ParamType是传值
f1(name); // 数组退化为指针const char*，T推导为const char*

template<typename T>
void f2(T& param); // ParamType是引用
f2(name); // 保持数组类型，T推导为const char[13]
```

```c++
// 以下这个小函数巧妙的利用了该规则保持数组类型的特点
template<typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) noexcept {
    return N;
}
// constexpr表示在编译期就是用其结果，将在第15条说明
// noexcept将使编译期生成更好的程序，将在第14条说明
```

* 引数为函数时，规则跟数组是一样的也存在退化的情况。

* 总结下，模板函数的类型推导的三种情况可以简单归纳为：

  * （情况二）如果引数是通用引用且是左值，则T和ParamType是一样的类型，都是引用；否则
  * （情况一）如果引数是普通引用，则视同非引用；否则
  * （情况三）就是ParamType是传值的推导，expr中的引用、const、都要去掉。