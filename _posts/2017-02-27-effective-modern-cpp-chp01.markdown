---
layout: post
title:  "《effective modern c++》笔记 导读+第1章 类型推导"
date:   2017-02-27 13:06:21 +0800
---

> echo: 本书是作者继<effective c++>和<more effective c++>之后的又一力作，着重c++11和c++14标准中新增的大量特性。 本书目前只有台湾繁体中文版(淘宝有售，有些术语叫法跟大陆不太一样)，网上也有在线的[简体中文翻译](http://www.kancloud.cn/kangdandan/book/169966)。

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

> `auto`和`decltype`的使用正在越来越广泛，使用类型推导写出的代码更能适应变化，更易维护。只要在一个地方修改类型，通过类型推导就可以使改动扩散到程序其他地方。但是，代码不能直观呈现类型推导的结果，书写正确代码的难度也变高了。为了更有效率的书写代码，本章提供了所有C++开发人员需要了解的类型推导知识。

> 阅读本章前，应当对`auto`和`decltype`关键字有简单基本的概念了解，本书不会对这些基本概念做讲解，而是深入探讨类型推导的规则。

## 1. 了解模板类型推导

* 在了解推导规则前，先看下模板函数的声明及调用的模型：

```c++
template<typename T>
void f(ParamType param); // param是参数，它的类型是ParamType

f(expr); // expr是引数，由expr推导出T和ParamType
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
  1. 如果expr是引用，则去掉引用。（因为传值，所以参数是个拷贝，肯定不会是引用）
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

```c++
void foo();

template<typename T>
void f1(T param); // ParamType是传值
f1(foo); // param是指针void(*)()，可以改为指向其他函数的指针

template<typename T>
void f2(T& param); // ParamType是引用
f2(foo); // param是引用void(&)()，使不能修改值的
```

* 总结下，模板函数的类型推导的三种情况可以简单归纳为：

  * （情况二）如果ParamType是通用引用，且引数是左值，则T和ParamType都是引用；否则
  * （情况一）如果ParamType是指针或引用，引数视为非引用，并进行模式比对；否则
  * （情况三）那ParamType一定是传值，引数视为非引用、非const、非volatile，做模式比对。

* echo：有一个简单的规则判断自己对类型推导的判断是否正确，我们不考虑`通用引用`的情况（我们极少用），如果引数类型是A，则：

  1. 推导出来的T永远不可能是A类型的指针或引用。因为模板参数必须自己写明是指针还是引用；
  2. 而且，如果模板参数是传值，则推导出来的T永远不可能是引用和const。因为传值得到的参数是一个拷贝，有自己的地址，一定是可以修改的。

## 2. 了解auto类型推导

* auto的类型推导跟函数模板的推导是一样的，除了一个特殊的例外。

* 模型：对于`const auto x = 27`，`const auto`就对应函数模板的ParamType，`auto`就对应T，而`27`就对应expr引数。

* echo：因此，根据第1条中我总结的规则，不考虑`通用引用`的情况，对于引数类型A，auto推导出的类型永远不可能是A的指针或引用；对于传值的情况，auto推导出的类型永远不可能是引用和const。例如这几个有意思的例子：

```c++
// 例子1：
const int x = 27;
auto ax = x;
// 很多同学会认为auto是const int，这是错误的。
// 根据规则，这是传值的情况，auto退导出的类型不可能是const，是int。
// 不信的话可以试试看，会发现ax是可以被赋值的，不是const类型。
// 如果要使ax的类型是const int，要这样写： const auto ax = x;

int y = 27;
int& ry = y;
auto ay = ry;
// 同样的，auto推导出的类型不是int&，而是int。
// 不信的话可以试试看，会发现ay和ry的地址是不同的。
// 如果要使ay的类型是int&，要这样写： auto& ax = x;
```

> echo：这告诉我们一个简单代码书写规则，如果要让ax、ay的类型是const或引用，一定要在auto的左边加const，右边加引用符号(&)。

* 本条最开始说的特殊的例外，是在使用c++11的统一初始化功能时，以下代码中`x2`的类型不是int，而是`std::initializer_list<int>`：

```c++
int x1 = {27};
auto x2 = {27};
auto x3{27}; 
```

> 根据C++17草案[N3922](http://www.open-std.org/JTC1/SC22/WG21/docs/papers/2014/n3922.html)，x3的类型会被推导为int，而不是`std::initializer_list<int>`，各编译器(2017年3月)目前会有不同的表现。

* 上面的规则是auto推导的特殊规则，并不适用于函数模板的推导：

```c++
template<typename T>
void f1(T param);
f1({1}); // 编译错误，无法推导

template<typename T>
void f2(std::initializer_list<T> param);
f2({1}); // 编译成功！
```

## 3. 了解decltype

* 在`operator []`的使用中会有些特殊使用技巧。接下来会用一个例子来展示。

* 对于T的容器，`operator []`通常是返回T&。假设在C++11中编写了一个`验证权限，获取容器指定下标元素引用`的模板函数：

```c++
template<typename Container, typename Index>
auto authAndAccess(Container& c, Index i)
  -> decltype(c[i]) // trailing return type，此语法允许我们在参数列表之后指定返回类型
{
  doAuthCheck();
  return c[i];
}
```

> 这样的函数模板比C++98的写法优势在于，我们不需要知道Container容器中的元素类型。无论何种容器，只要支持下标操作符就可以使用模板。代码更加通用，当然也敲字也敲的少。

* C++14中，上面的代码可以更加精简为：

```c++
template<typename Container, typename Index>
auto authAndAccess(Container& c, Index i)
{
  doAuthCheck();
  return c[i];
}
```

* 但实际上，我们调用函数时，会遭遇编译失败：

```c++
authAndAccess(some_list, 5) = 10; //尝试得到第五个元素的引用并进行赋值
```

* 失败的原因在于auto的推导规则，我们前两节总结果规律了，在传值的情况下，auto的推导不可能得到引用类型，因此上面的函数里auto得到的一定是一个值拷贝。因此赋值操作一定是无法编译通过的。

* 我们简单的修改代码就可以正常使用了：

```c++
template<typename Container, typename Index>
auto& authAndAccess(Container& c, Index i)
{
  doAuthCheck();
  return c[i];
}
```

* 但是，并不是所有的`opeartor []`都返回引用。比较特殊的例子有`std::vector<bool>`，它的`operator []`会返回一个对象（第6条会做详细说明）。我们的函数在使用`std::vector<bool>`时是无法编译通过的。

* 通过使用`decltype(auto)`可以让我们的函数变得更加通用：

```c++
template<typename Container, typename Index>
decltype(auto) authAndAccess(Container& c, Index i)
{
  doAuthCheck();
  return c[i];
}
```

* `decltype(auto)`这个搭配看起来很奇怪，但实际上是合理的：auto表示使用类型推导，decltype表示使用decltype的规则来进行推导。这样就我们就能让返回值的类型始终与c[i]的类型保持一致。

* 接下来，本节讨论了很多要对上面函数的参数c传入右值的情况，因为我们极少使用右值引用，本部分内容不做笔记。

* decltype的表现几乎总是符合预期的，很少有意外的情况。但是这里有特别规则要注意：decltype对变量名使用返回变量的类型，但是对左值表达式使用会得到引用类型。这是一个潜在的坑，例如，对于`int x = 27;`的变量x，`decltype(x)`会得到int，而decltype((x))会得到int&。这是因为`(x)`是一个左值表达式，而不是变量名。

* 由于上面提到的这个特别规则，以下两个函数使用了decltye(auto)来变得更加通用，但两个函数返回语句的写法却导致了行为的极大改变：

```c++
decltype(auto) f1()
{
  int x = 0;
  return x; // f1返回int
}
decltype(auto) f2()
{
  int x = 0;
  return (x); // f2返回int&
}
```

> 对于decltype(auto)的使用必须极为小心，一些看似无足轻重的改动可能会极大的改变代码行为。但这不是常态，绝大多数情况下decltype都是符合预期的，不应因噎废食。

## 4. 学习如何查看推导的类型

* 通过IDE。但是IDE对于复杂的类型能够提供的帮助就比较有限。

* 借住编译期编译错误信息。

```c++
TD<decltype(x)> xType;

// 由于TD模板未定义，编译时会产生错误信息，包含了x的类型信息。
```

* 运行时使用typeid输出类型信息。

```c++
cout << typeid(x).name() << endl;

//输出i表示int，输出PK则表示[Pointer to konst(const)]。
//输出PK6Widget，表示指向Widget的指针。6表示类名字Widget有6个字符。
```

* typeid给出的类型信息不可靠，可能会忽略引用、const信息。IDE有时也无法给出可靠的类型。

* `Boost::TypeIndex`可以产生正确的类型信息。