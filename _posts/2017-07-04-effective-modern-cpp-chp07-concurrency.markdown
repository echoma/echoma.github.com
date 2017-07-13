---
layout: post
title:  "《effective modern c++》笔记 第7章 并行API"
date:   2017-07-04 13:25:21 +0800
---

* 目录
{:toc}

# 第七章

* c++对并行的支持是一大进步，而且也只是个开始。

* 标准库中有两个`future模板`：`std::future`和`std::shared_future`，大部分情况下这两者的差异不重要，书中讲到的`future`通常同时表示这两者。

## 35. 尽量用任务式的程序设计取代线程式的设计

* 假如要非同步执行doAsyncWork函数，有两种选择：

  1. 使用线程式（thread-based）的设计：`std::threaad t(doAsyncWork);`
  2. 使用任务式（task-based）的设计：`auto fut = std::async(doAsyncWork);`

* 通常任务式的设计有更多好处：

  1. 有返回值。线程式设计是没办法直接获取返回值的。而`std::async`提供get函数轻松获取返回值。
  2. 支持异常捕获。通过get函数可以取得异常。而线程式的设计一旦异常就导致程序终止（通过调用std::terminate）。

* 几个概念要区分清楚：

  1. 硬件线程（hardware thread）：现代的计算机CPU在每个核心上都提供一个以上的线程。
  2. 软件线程（software thread）：操作系统进程内部的线程，排在硬件线程上执行。一旦阻塞（如果等待IO操作、等待互斥量），硬件线程就会切换到其他未阻塞的软件线程来执行。
  3. std::thread：这是C++的类，是底层软件线程在C++的句柄（handle）。当然，一个std::thread也可能不代表任何底层软件线程（而是代表null），可能是因为处于刚刚被构造的状态、曾经作为移动操作的来源等。

* 多线程编程会有两个棘手情况：

  1. 软件线程资源耗尽：软件线程是有限资源，操作系统是限制了线程数量的，如果超出会抛出`std::system_error`异常。
  2. 过度订阅（oversubscription）：线程数量太多，一个线程被系统不停地调度到不同的CPU核心上运行，线程上下文切换成本变得很高。

* `std::async`在面对上述两个棘手问题时是有做了一些工作的：`std::async`不保证会建立新的软件线程。当上述两个棘手情况发生时，计算工作会被调度到`调用get或wait操作`的线程。当然，这可能会对调用get或wait操作的线程造成影响，如果这个线程是负责处理GUI的，那可能会导致界面失去响应。第36条会讲解启动策略来配置这个行为。

* 当然，`std::thread`也有其适用场景：

  1. 需要存取底层线程实现API：有时候需要操作底层的pthread或者Windows threads，因为他们提供了更丰富的功能。`std::thread`会提供`native_handle`成员函数。
  2. 需要针对硬件环境、软件逻辑优化线程的使用：软件逻辑比较特殊，或者硬件特性比较特殊。这种情况下需要开发者自己来调整调度方法。
  3. 要实现标准库没有提供的线程特性：比如线程池。

## 36. 有必要非同步时就指定std::launch::async

* `std::async`有两种启动策略（launch policy）:

  1. std::launch::async：表示计算必须以非同步的方式在另外一个线程执行。
  2. std::launch::deferred：表示计算会延迟(defferred)执行，会在返回的future调用get或者wait时才执行。调用者线程会被阻塞到计算完成。

* std::async的默认启动策略，是以上两种方式的`或`:

```c++
// 以下两个future具有相同的启动策略
auto fut1 = std::async(f);
auto fut2 = std::async(std::launch::async | std::launch::deferred, f);
```

* 当我们在线程t上执行`auto fut = std::async(f)；`时，我们使用了默认的启动策略，我们会面临以下问题：

  1. 无法预测f是否会跟当前线程t并行执行。f可能会延迟执行。
  2. 无法预测f是否会在执行fut的get和wait的线程里执行。尤其是当在t里执行get或wait时，更是没办法预测。
  3. 无法预测f是否被执行。因为fut的get和wait可能根本没有被调用。
  4. 由于第1、2点，有时不太适合使用`thread local`变量。这会使用线程本地存储(thread local storage, TLS)，但我们无法预测f执行时会存取那个线程的变量。
  5. 由于第3点，`wait_for`和`wait_until`这类看起来可以等待到f运行结束的调用，实际上可能永远不会等到。

* 要想知道一个future对象是否是deffered，没有直接的接口获取，只能通过`wait_for(0)`的返回码是否是`std::future_status::defurred`来判断。

* 如果想要保证f一定是与调用线程异步执行的，就指定启动策略为`std::launch::async`。

## 37. 确保std::thread在所有路径都是unjoinable

* 所有std::thread的状态不是joinable(可链接)就是unjoinable(不可连接)。

* `joinable`表示这个std::thread对象关联了一个底层线程。

* 为什么我们要关注是否可连接的状态呢？这一点非常重要：**对一个joinable的std::thread对象调用析构函数会导致程序崩溃退出**。

> echo：我亲自试了一下，确实是会导致coredump。
> 　　　代码里就是简单的声明了一个thread对象（当然要指定其执行函数）。屏幕上会输出：
>　　　　terminate called without an active exception
>　　　　[1]    11661 abort (core dumped)  ./a.out

* std::thread的状态是unjoinable的情况列举如下：

  1. 使用默认构造函数构造的std::thread对象。因为它没有要执行的函数，因此一定是不可连接的。
  2. 曾经作为移动操作来源的std::thread对象。因为底层的线程已经被移动到了其他对象。
  3. std::thread已经被join了，join结束后的std::thread对象已经执行完毕，不在对应底层的线程。
  4. std::thread已经被detach了，这会切断std::thread跟底层线程的关系。

> echo：上面的这几种情况描述在[std::thread::~thread()文档](http://en.cppreference.com/w/cpp/thread/thread/~thread)也有描述。

* 只要不是以上情况的，均视为joinable。

> echo：因此我们要注意，一个thread对象如果没有被join过，即使底层线程已经执行完毕了，thread对象也仍然是joinable的。因为线程是异步执行的，系统并不知道你将来是否会调用join，所以就要求要么join过，要么detach过，否则就是joinable的。

> 我们可能会想，为什么不让thread对象默认就join或者默认就detach。
> 书中也给出了标准委员会的考虑：
> 　　　1. 默认join：不符合一般直觉。这会导致声明一个线程对象直接就阻塞了。
> 　　　2. 默认detach：有些场合是需要join()的。比如，线程里用到了调用者代码里的栈变量，这时调用者需要等待线程执行完毕才能弹出栈。
> 标准委员会基于这些考虑，决定不能默认join和默认detach。

* 我们在写代码的时候，可能不小心就会发生调用joinable对象的析构函数的情况：

```c++
bool func()
{
    // 声明了一个线程对象，线程的工作就是打印一句话。
    // 这个线程对象是joinable的。
    std::thread t([](){ cout << "do some work" << endl; });

    if (condition_satisfied()) // 如果条件满足，就join()等待线程执行完
    {
        t.join();
    }

    // 如果条件没满足，函数返回，线程对象的析构函数会被调用，这是程序会崩溃退出。
    // 一种正确的方案，是调用detach：
    //    if (t.joinable())
    //        t.detach();

    return false;
}
```

> 注意，上面这段代码的正解中，我们先判断了t.joinable()才进行了detach操作。这时因为：对unjoinable的thread对象调用detach会引发崩溃。
> 这里就会有个隐藏的问题：如果我们调用t.detach()前，有另外的线程也调用了detach()，就会引发崩溃。因此这里开发者自己要确认这里是否会存在并发调用的情况。

* 诸如continue、break、goto这类语句，都可能会引发这个问题。更隐蔽的，析构函数中对成员变量的析构顺序也可能会引发这个问题。例如成员变量中有个线程对象，该线程使用了其他成员变量，那开发者就要确保线程在其他成员变量析构前结束。

* 因此，虽然本章标题说要保证thread对象从定义到离开调用范围的所有路径都要保证unjoinable，但实际上实操起来是比较复杂的。