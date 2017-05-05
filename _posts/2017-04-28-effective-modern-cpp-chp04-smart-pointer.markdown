---
layout: post
title:  "《effective modern c++》笔记 第4章 智能指针"
date:   2017-04-28 13:45:21 +0800
---

* 目录
{:toc}

# 第四章

> c++98中的`auto_ptr`在c++11中已经纯粹是为了兼容老标准而存在了。

> 新增的三种智能指针适用性更好（`unique_ptr`,`shared_ptr`,`weak_ptr`），可用于容器内部，支持移动操作，而且语义更清晰。

> 原来使用`auto_ptr`的地方都应当义无反顾的使用新的`unique_ptr`，除非是为了兼容老标准。

## 18. 使用std::unique_ptr管理单一所有权的资源

* `unique_ptr`代表单一所有权，体积小(大部分情况和原指针同样大小)，速度快，不支持复制，只支持移动操作。

* `unique_ptr`默认使用delete删除对象，通过第二个模板参数可以传入`删除子`类型，指定删除操作。

* 可以直接通过类型转化转为`shared_ptr`类型。

## 19. 使用shared_ptr管理共享所有权的资源

* `shared_ptr`使用引用计数来判断对象是否应该被删除了，且其更新引用计数的操作内部都是用atomic来确保线程安全。

* `shared_ptr`内部有两个指针，因此体积是原指针的两倍（标准未定义实现方式，但几乎所有的标准库实现都使用了这种实现）。

  * 一个指针指向原始对象（即构造函数传入的指针）

  * 一个指针指向一个控制区块，该区块存放了引用计数、弱计数、自定义删除子。关于弱计数，在第21节会有进一步描述。

* 删除子不是`shared_ptr`的模板参数，而是存放在控制区块里的。也就说，同一类型的`shared_ptr`的多个对象可以拥有各自不同的删除子。这跟`unique_ptr`是不同的。

* 原则上，使用`shared_ptr`是一条**不归路**，一旦将某个原始指针交给`shared_ptr`管理，今后就不应该对原始指针进行操作了。尤其是用同一个原始指针构造多个`shared_ptr`对象，会导致引用计数紊乱。

## 20. 用std::weak_ptr取代可能悬置的std::shared_ptr类指针

* `weak_ptr`类似`shared_ptr`，但不会影响引用计数。`weak_ptr`必须使用`shared_ptr`对象来构造。

* 其`expired()`方法可以用来检查所代表的原始指针指向的对象是否已被释放。

* 使用`expired()`方法，通常是遵循先判断过期再使用的逻辑(例如`if expired() { delete wk_ptr; }`)，如此在多线程访问过程中可能跟原`shared_ptr`的操作存在竞争，比如导致多次释放。有两种方法解决此问题：

  * 使用`lock()`方法获取一个`shared_ptr`对象，如果该对象不是空，则可以使用`shared_ptr`进行线程安全的操作。

  * 使用`weak_ptr`对象构造一个`shared_ptr`对象，如果没有抛出异常，则可以使用`shared_ptr`进行线程安全的操作。

* 有些时候，需求使两个对象各自拥有指向对方的`shared_ptr`，使得引用计数永远不可能变为0，对象永远都不能得到真正的释放。此时将其中一个改为`weak_ptr`就能解决问题。

## 21. 尽量用std::make_unique与std::make_shared取代直接使用new

* 比起先new原始指针再赋值给new出来的智能指针的方式，直接使用make函数精炼很多，而且性能也更高。

* `make_unique`函数在c++14才引入，使用c++11的话就需要自己定义，这很简单(代码见书，使用了variadic template)。但注意不要污染std名字空间，以防今后升级到c++14后发生冲突。

* make函数无法自定义删除子，无法调用对象的大括号构造函数。

* 当`shared_ptr`控制区块里的引用计数是0时，原始对象被清楚，但控制区块却不一定被清除。控制区块里的弱引用计数记录了引用了该区块的`weak_ptr`数量。当弱引用计数也变成0的时候，控制区块才会被最后释放。因此同时使用`shared_ptr`和`weak_ptr`可能会碰到内存的释放比实际预想的要晚的情况。

## 22. 使用Pimpl Idiom时，将特殊成员函数定义在实现文件

* `Pimp Idiom`：用于缩短编译时间。将类的具体逻辑用另外的一个实现类来实现，该实现类的代码都放在`.cpp`文件里。这样实现类的声明和引用的头文件发生变化时，不会引起大批量的重新编译。

```c++
// 下面这个Widget类引用了Gadget类，Gadget类在gadget.h里定义的。


// 非Pimp Idiom的做法如下
#include "gadget.h"
class Widget {
public:
  Widget();
  ~Widget();

private:
  Gadget g1, g2, g3;
};


// Pimp Idiom的做法如下
// Widget.h
class Widget {
public:
  Widget();
  ~Widget();

private:
  struct Imp;
  Imp* pImp;
};
// Widget.cpp
#include "gadget.h"
Widget::Widget():pImp(new Imp()) { }
Widget::~Widget() { delete pImp; }
struct Widget::Imp {
  Gadget g1, g2, g3;
};


// Pimp Idiom的智能指针做法如下
// Widget.h
class Widget {
public:
  Widget();
  ~Widget(); // 必须有析构函数，否则默认析构函数找不到Impl的实现回编译出错

private:
  struct Imp;
  std::unique_ptr<Impl> pImp;
};
// Widget.cpp
#include "gadget.h"
Widget::Widget():pImp(std::make_unique<Impl>()) { }
Widget::~Widget() { } // 析构函数在.cpp中，就可以找到Impl的实现
Widget::~Widget() = default; // 或 要求使用默认的析构函数
struct Widget::Imp {
  Gadget g1, g2, g3;
};

```

* 注意上面代码最后一段智能指针的做法，析构函数是必须在`.cpp`里进行实现或在`.cpp`里使用`=default`的。因为`unique_ptr`的删除子是模板参数，默认的删除子是需要完整的类型信息的。这也是理所当然的，释放对象的时候一定要知道对象的具体类型，才能准确的调用到对象的析构函数。如果上面的代码里没有声明析构函数，编译器会在解析`Widget.h`时就开始生成默认析构函数，并查找Impl类的完整类型信息，但`.h`里没有这个类型信息，因此会编译出错。

* `shared_ptr`跟`unique_ptr`类似，但其删除子不是模板参数，并不要求有完整的类型信息。上面的代码如果换做使用`shared_ptr`是不需要声明析构函数的。关于`shared_ptr`和`unique_ptr`的这点区别，[这里](https://howardhinnant.github.io/incomplete.html)有详细的列表，还有这篇[来自stackoverflow的帖子](http://stackoverflow.com/questions/6012157/is-stdunique-ptrt-required-to-know-the-full-definition-of-t)也有讨论。

* echo：为了证明确实可以实现类似shared_ptr这种不需要完整类型信息就可以实现delete的模板类，特意试了一下，写了个自己的指针类，果然可以。下面这段代码可以编译通过并正常运行，不晓得是C++11的什么特性，对于模板来说类型的具体信息可以比使用类型实例化模板更晚出现也没问题。

```
// test.h
#include <memory>
using namespace std;

// 自己的指针类
template<typename T>
class MyPtr
{
public:
  using Deleter = void(*)(T* pt); // 删除子类型

  MyPtr():t(nullptr)
  {
    dlt = [](T* pt) { delete pt; }; // 给删除子赋值
  }

  ~MyPtr()
  {
    if(nullptr!=t) 
    {
      dlt(t);
      t=nullptr; 
    }
  }

  void reset(T* pt)
  {
    t = pt;
  }

  T* t; // 指向原始指针
  Deleter dlt; // 删除子
};


// test.cpp
struct Foo; // 前向声明，非完整类型

MyPtr<Foo> foo; // 没有Foo类型的完整信息，一样可以进行实例化

struct Foo
{
  int a;
};

int main()
{
  foo.reset(new Foo());
  return 0;
}
```
