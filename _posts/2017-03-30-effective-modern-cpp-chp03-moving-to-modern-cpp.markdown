---
layout: post
title:  "《effective modern c++》笔记 第3章 迈入现代C++"
date:   2017-03-30 13:45:21 +0800
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

  * 容器初始化，只能用`{ }`大括号形式：`std::vector<int> v{1,2,3}`。

  * C++11的非静态成员初始化不能使用小括号：

  ```c++
  class Foo {
  private:
    int x{0}; // OK
    int y = 0; // OK
    int z(0); // 编译错误
  };
  ```

  * 无法赋值的对象（如std::atomic，第40条会介绍），不能使用=号初始化：

  ```c++
  std::atomic<int> x{0}; // OK
  std::atomic<int> y(0); // OK
  std::atomic<int> z = 0; // 编译错误
  ```

* 通过上面的场景可以发现，`{ }`是应用范围最广的初始化形式，被称为`大括号初始化列表(braced-init-list)`语法，这是c++11引入的`通用(uniform)`初始化形式，适用于所有场景。

* `{ }`形式还有与其他形式与众不同的地方：

  * 还可以避免`向下转型(narrowing conversion)`：

  ```c++
  int a {0.2}; // 编译错误
  double x = 0.2;
  int b {x}; // 编译错误
  int c (x); // OK
  int d = x; // OK
  ```

  * C++会把所有能视为声明的东西看做是声明，这有时候有副作用：

  ```c++
  Widget w1(10); // 用10构造对象w1
  Widget w2(); // 本意是像使用默认值构造对象w2，但实际上被视为声明了函数w2
  Widget w3{}; // 用默认值构造对象w3
  ```

* 通过上面的阐述我们发现，使用`{}`形式好处多多。那为什么本节的标题不直接叫做`优先使用{}初始化语法`呢？因为有几个特殊情况：

  * `{}`会被编译器优先推导为`std::initializer_list`类型，如果我们使用`auto`关键字声明对象，那对象的类型会被推导为`std::initializer_list`类型。这一点我们在第2条有说过，我们还提到了草案N3922。因此，在使用`auto`声明变量时，不建议使用`{}`形式初始化。

  * 如果某个类，支持`std::initializer_list`参数的构造函数，那在使用`{}`形式初始化该类的对象时，编译器优先使用`std::initializer_list`参数的构造函数。

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

## 9. 尽量用别名声明取代typedef

* C++11提供了`别名声明(alias declaration)`，之前`typedef`的功能可以这样改写：

```c++
typedef void (*FP)(int, const std::string&); // typedef
using FP = void (*)(int, const std::string&); // 别名声明
```

* 上面例子并不能看到别名声明比typedef好在哪里。别名声明的优点是：它声明的类型是支持模板的（别名模板）。有过设计大量模板经验的开发者都曾经历的痛苦过程就是：通过一个模板内部的typedef定义，来定义另一个模板的内部定义，这类template metaprogramming的代码是较晦涩的(echo：后台ut库的大量模板代码就有大量的这种场景)。使用`别名声明`，代码会清晰很多：

```c++
// 以下例子首先定义了一个使用自定义allocator的链表，然后再定义一个Widget类使用这个链表。

// 首先看下typedef的写法
template<typename T>
struct MyAllocList {
  typedef std::list<T, MyAlloc<T>> type;
};
template<typename T>
class Widget {
private:
  typename MyAllocList<T>::type list; // 定义成员，使用了这个链表类型
};

// 再看下使用别名声明的代码
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;
template<typename T>
class Widget {
private:
  MyAllocList<T> list; // 不需要使用typenmae和::type
};
```

* c++14开始，标准库中也开始大量使用别名样板了。（标准委员会太晚认识到别名样板的好处，所以c++11中还没有大量使用）。

## 10. 尽量使用scoped enum，而非unscoped enum

* 开发者们都意识到enum中列举项(枚举子,enumerator)的名称的作用范围很大，在enum可见范围内是不能出现其他相同名称的东西的(变量、函数都不行)：

```c++
enum Color { black, white, red };
auto white = false; // 编译错误，white已经声明过了
void red() { } // 编译错误，red已经声明过了
```

* 这其实是命名空间的命名污染问题。在定义枚举类型时，这个问题就迫使我们要给枚举项取一个不会跟整个命名空间内其他变量、函数冲突的名字。而使用`scoped enum`可以完美解决这个问题：

```c++
enum class Color { black, white, red}; // 枚举类！这其实是一个类。
auto white = false; // 编译成功！
void red() { } // 编译成功！
Color c = Color::black; // scoped enum必须使用Color::指明枚举类型
```

* `scoped enum`还有其他几方面的好处：

  * 强类型。由于`scoped enum`的表现更像是类，不能用整数等数字类型直接赋值，不能用不同枚举类型的值互相赋值，不能用枚举类型跟数字进行比较。这使得我们的代码安全了很多。如果实在要跟数字进行赋值和比较，必须在代码中显式的使用`static_cast<>`语句：

  ```c++
  enum class Color {black, white, red};
  Color a = 1; // 编译错误，不能用数字初始化或赋值
  auto c = Color::black;
  c = static_cast<Color>(5); // 可以使用static_cast来用数字赋值
  if (static_cast<int>(c)>1.2) // 使用static_cast来作比较
    cout << "Bigger" << endl;
  ```

  * 可以前向声明(forward-declaration)。

  ```c++
  enum class Status; // 前向声明
  void someFunc(Status s); // 编译成功
  ```

    > 这使得我们的代码可以减少对头文件的依赖程度。在无法做前向声明时，假设A.h定义了枚举，B.h里的某函数声明的参数使用了该枚举，C.h使用了B.h里的其他函数，则当枚举的定义发生变化时，包含了A.h/B.h/C.h的所有代码都需要重新编译。如果有了前向声明，B.h就不需要包含A.h了，只需要自己前向声明就可以了，当枚举的定义发生变化时，只有包含了A.h的源代码文件需要重新编译。

* C++11中我们可以指定枚举类型在底层使用的存储空间。默认是int。指定方法如下：

  ```c++
  enum class Status; // scoped enum 的前向声明
  enum class Status : std::uint32_t; // 指定底层类型的前向声明
  enum class Status : std::uint32_t {
    good, failed, incomplete, autidted
  };
  enum Color; // unscoped enum 的前向声明
  enum Color : std::uint8_t; // 指定底层类型的前向声明
  enum Color : std::uint8_t {
    black, white, red, green
  };
  ```

## 11. 尽量用deleted函数取代private无定义函数

* 在C++98中，如果我们要禁用拷贝构造函数，通常我们会将拷贝构造函数声明为`private`，且不编写他的实现代码。这样所有拷贝对象的地方，都会在链接时发生错误。

* c++11中新增了`deleted function`特性：

```c++
class SomeFoo {
  public:
    SomeFoo(const SomeFoo&) = deleted;
    SomeFoo& operator=(const SomeFoo&) = deleted;
}
```

* 该语法看上去跟之前`private`声明方式只是风格不同，其实差异很大。`deleted function`有以下几方面的优势：

  * 编译期就会产生错误，而且错误信息更加清晰。
  * 即使是成员函数、友类也无法调用。
  * 不仅仅类成员函数可以用，所有函数都可以标记为`deleted function`。
  * 通过使用该特性，可以禁止函数重载的某种形式，可以禁止模板的某些类型的特化。

## 12. 将重写函数宣告为override

* 子类对基类方法的重写，需要满足以下条件：

  * 基类函数必须是virtual
  * 基类与子类函数的名称必须相同（析构函数除外）
  * 基类与子类函数的参数类型必须相同。
  * 基类与子类函数的const标记相同。
  * 基类与子类函数的返回值类型和异常定义必须相同。
  * c++11新增：基类与子类函数的引用限定符必须相同。

* 根据以上规则，下面这段代码中，子类没能重写任何父类的函数：

```c++
#include <iostream>
#include <fstream>
#include <cstdint>
#include <typeinfo>
using namespace std;

class Base {
public:
  virtual void mf1() const { cout << __FUNCTION__ << " @ " << __LINE__ << endl; }
  virtual void mf2(int x) { cout << __FUNCTION__ << " @ " << __LINE__ << endl; }
  virtual void mf3() & { cout << __FUNCTION__ << " @ " << __LINE__ << endl; }
  virtual void mf4() const { cout << __FUNCTION__ << " @ " << __LINE__ << endl; }
};

class Derived: public Base {
public:
  virtual void mf1() { cout << __FUNCTION__ << " @ " << __LINE__ << endl; }
  virtual void mf2(unsigned int x) { cout << __FUNCTION__ << " @ " << __LINE__ << endl; }
  virtual void mf3() && { cout << __FUNCTION__ << " @ " << __LINE__ << endl; }
  void mf4() const { cout << __FUNCTION__ << " @ " << __LINE__ << endl; }
};

int main()
{
    Base* b = new Derived();
    b->mf4();
    return 0;
}
```

* 上面这段代码在大多数编译器都不会产生任何警告信息(echo:`g++5.4`开启`Wall`选项不会有任何告警信息)。而这样的代码可能根本不是开发人员的意图，开发人员可能想要重写。将函数声明为`override`，可以让编译器在未能成功重写的函数处给出错误。

* `override`还有一个好处。我们修改基类的函数声明后，影响的范围在c++98时代是很难评估的，我们不知道哪些子类重写了该函数，只能靠全面的测试来排除。而有了`override`，编译器将为我们指出受影响的子类。

* TODO：本章还有关于左值、右值的讨论，还没太看懂，以后看了右值+move语法后来补充。

## 13. 尽量用const_iterator取代iterator

* c++98中的`const_iterator`只有形式上的支持，实际编程中难以建立、使用方式十分受限。在c++98中，我们不能从`iterator`获取`const_iterator`，容器也缺乏能够产生`const_iterator`的方法。可以说，在c++98中，`const_iterator`并不实用。

* c++11弥补了这些不足，但是仍然缺少`非成员方式的`泛型的cbegin和cend函数。c++14进一步补充了这些函数。

## 14. 将不会产生异常的函数声明为noexcept

* c++98中，异常规格(exception specification)的性能不稳定。函数的异常规格必须记录所有函数会跑出的异常类型，一旦函数的实现发生变动，异常规格也可能要随之修改。这类修改可能破坏依赖原有规格的代码，开发人员难以维护。

* c++11中建立了新的共识：只关心函数是否会抛出任何意外。因此c++11中引入了noexcept表示“保证不会抛出任何异常”。

* noexcept还有另一个好处：编译器能够产生更优的目标码。


```c++
int f(int x);          // 不指定。较少优化。
int f(int x) throw();  // c++98方式。较少优化
int f(int x) noexcept; // c++11方式。较多优化。
```

* 注意，虽然noexcept带来了性能优化，但需要谨慎使用：
  1. 愿意长期维持noexcept的函数才应该声明为noexcept。否则函数调用者的异常规格可能会随着函数的异常规格而变化，这很难维护。
  2. 不应当为了给函数加上noexcept二调整函数的实现，这是反客为主的做法。大多数函数应当都是异常中立(exception-neutral)的。

* delete操作符、析构函数默认都是noexcept的。

## 15. 尽可能使用constexpr

* constexpr可以用来修饰对象，也可以修饰函数。

* constexpr比const提供了更多的特性：
  1. constexpr对象必须在编译期就能确定数值。
  2. constexpr函数如果无法在编译期确定数值，其行为就像普通函数一样。

* c++11中，constexpr函数只能有一条语句，且必须是return语句。而c++14中不需要了。

* c++11中，类成员函数可以使constexpr的，甚至构造函数也可以(用于构造可以在编译期确定数值的对象)。

## 16. 让const成员函数具备多线程安全

* 声明为const的成员函数可能在多线程并发访问环境下变得不const，尤其是const成员函数内部修改了声明为mutable的成员变量时。

* 除非很确定不会在并发环境中使用，否则都应当尽量确保const成员函数具备多线程安全性。(echo：有点异议)

* 使用std::atomic会比mutex有更好的性能，不过适用范围会小些。

## 17. 了解特殊成员函数的生成

* 特殊成员函数，是指c++会自动生成的成员函数。c++98中包括默认的构造函数、析构函数、拷贝构造函数、(拷贝)赋值运算符。c++11中因为引入了移动操作，又新加入了默认的移动构造函数和移动赋值运算符。

* c++98中的`Rule of Three`在c++11仍然有效：在一个类里，两种拷贝操作(构造函数和赋值运算符)和析构函数必须同时声明。

* 移动操作的默认函数的自动生成需要满足三个条件：
  1. 没有声明拷贝构造函数和(拷贝)赋值运算符。
  2. 没有声明移动构造函数和移动赋值运算符。
  3. 没有声明析构函数。
  > 主要是因为以上三类函数都涉及资源的管理，一旦我们自己声明了这三类函数，默认的移动操作就很可能是有问题的了。

* 使用`=default`可以要求编译器使用默认的特殊成员函数实现。

* 默认的移动操作和拷贝操作是互斥的。一旦我们主动声明了移动操作，那编译器就不会生成默认的拷贝构造函数和(拷贝)赋值运算符。一旦我们主动声明了拷贝操作，则默认的移动操作函数也不会生成。

* 关于成员函数模板对特殊成员函数的影响，将在条目24中阐述。

