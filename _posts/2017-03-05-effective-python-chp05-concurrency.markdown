---
layout: post
title:  "《effective python》笔记 第5章 并发与并行"
date:   2017-03-05 16:47:21 +0800
---

> 并发(concurrency)是说计算机似乎在同一时间做着很多不同的事。假如某台电脑只有1个CPU，操作系统在各程序件切换，这种交错执行的方式造成了一种假象，好像这些程序在同时运行。
> 并行(parallelism)是计算器确实在同一时间做着好几件事。具备多个CPU核心的计算机有此能力。
> python编写并发程序比较容易，但是要使程序真正以平行的方式运行却相当困难。

## 36. 用subprocess模块来管理子进程

* python有很多运行子进程的方法，最简单且好用的进程管理模块是subprocess。
* 介绍了poll()和communicate()等常用方法。

## 37. 可以用线程来执行阻塞式I/O，但不要用它做并行计算

* Python的官方实现CPython有全局解释器锁(GIL)，用来避免系统的抢占式多线程的线程切换导致解释器处于错误的状态。这使得python的线程实际上是无法实现并行的。
* 在多核计算机上用python多线程来执行任务，最终的耗时比单线程还要长。
* Python的多线程适合用来做阻塞式I/O，使得我们可以在做阻塞操作时进行一些其他计算。这种场景下，python的多线程是可以优化体验和显著减少耗时的。

## 38. 在线程中使用Lock防止数据竞争

* 了解全局解释器锁后，不要认为编写python多线程代码就不需要锁了。真相并非如此，GIL不会保证开发者的代码不被其他线程打断，GIL只会保证单条python字节码指令不被打断。
* Python内置的threading模块中有个Lock类，实现了互斥锁。

## 39. 用Queue来协调各线程之间的工作

* 本节讲了一种设计思路，把工作的各个步骤拆分为独立的工作，对不同的步骤使用不同的队列和工作线程来进行处理。这其实是典型的生产者-消费者模型的应用。
* 内置的queue模块的Queue类有个get方法，在队列中无数据时可以阻塞，可以避免频繁的查询工作队列的状态。它还有其它特性，可以让开发者构造出更强健的代码。

## 40. 考虑用协程并发的运行多个函数

* 线程有很多缺点：需要避免数据竞争、占用内存较大（约8M）、启动开销大，而协程可以避免这些问题。

* python的协程本质上是对生成器的一种扩展（第16条有讲过生成器）。启动生成器的开销跟函数调用的开销相仿，内存占用不到1K。生成器只是在模拟并行，使程序看上去在同时运行多个函数。

* 示例代码：

```python
def my_coroutine():
    while True:
        received = yield
        print('Received:', received)

it = my_coroutine()
next(it) # 使生成器运行到yield处并暂停
it.send('First') # 使yeild返回字符串‘First’，然后再次循环至yield处并暂停
it.send('Second') # 使yeild返回字符串‘Second’，然后再次循环至yield处并暂停

>>>
Received: First
Received: Second
```

* python3具有`yield from`语法，可以让其他生成器的返回值触发yield返回：

```python
def second_coroutine():
    a = yield from first_coroutine()
    return a
## first_coroutine耗尽并返回后，a就会得到这个返回值。
```

* python2的生成器还不支持`return`返回值，如果要向外传输数据，通常需要抛出异常，通过异常捕获获得数据。