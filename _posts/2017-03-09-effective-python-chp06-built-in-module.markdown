---
layout: post
title:  "《effective python》笔记 第6章 内置模块"
date:   2017-03-09 13:13:21 +0800
---

> Python内置了很多有用的程序包。某些程序包跟Python的习惯用法紧密接合，实际上已经成了语言规范的一部分。

## 42. 用functools.wraps定义函数修饰器

* 修饰器可以在某个函数执行之前或之后分别运行一些别的代码。开发者可以使用修饰器修改原函数的输入参数及返回值，达到约束语义、调试等目标。请自行了解修饰器(decorator)的概念。

* 但调用使用了修饰器的函数时，会发现函数名变成了修饰器的名字，这时可以利用内置的functools.wraps来修饰。

```python
# 该修饰器可以打印函数的名称和返回值
def trace(func):
    # 使用了内置的functools.wraps来修饰指定函数
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print('%s -> %r' % (func.__name__, result))
    return wrapper

# 函数some_func()被定义为使用修饰器trace
@trace
def some_func():
    return 1

>>> some_func()
some_func -> 1
```

## 43. 使用contextlib和with语句来改写可复用的try/finnally代码

* Python内置的很多类对with语句进行了支持，编写代码更加精简。例如下面的代码改写：

```python
lock = Lock()

# 复杂的版本
try
    lock.acquire()
    print('Lock is held')
finally:
    lock.release()

# 精简的版本
with lock:
    print('Lock is held')
```

* 有大量的重复模式可以使用with语句来精简，比如“打开文件，使用文件句柄进行文件操作，最后关闭句柄”、“进入某函数调低日志级别，函数打印大量调试日之后，在函数退出前调高日志级别”。

* 如果我们要自定义with行为，按照python的标准方式要定义新的类，并实现\_\_enter\_\_和\_\_exit\_\_方法。，内置的contextlib模块提供了contextmanager修饰器，让我们可以非常轻松的让某种代码模式支持with语句：

```python
# 我们把上面提到的日志级别的模式改写为支持with语句
@contextmanager
def log_level(level, name)
    # try语句之前的这三行会在with块中代码执行前运行
    logger = logging.getLogger(name)
    old_level = logger.getEffectiveLevel()
    logger.setLevel(level)
    try:
        # 此处将会执行with块中的代码
        # 返回的logger作为with语句中as得到的值
        yield logger
    finally:
        # with块代码结束后会运行这里的代码
        logger.setLevel(level)

with log_level(logging.DEBUG, 'my_log') as my_logger:
    do_somthing()
    my_logger.debug('print some debug log')
```

## 44. 用copyreg实现可靠的pickle操作

* 内置的pickle模块可以非常方便的对python对象进行序列化和反序列化。但是，如果类新增了某个属性，使用旧版本类对象序列化得到的字节流，再反序列化为新版本的类对象时，将会缺少这个新属性。

* 内置的copyreg模块可以注册序列化和反序列化函数，自定义序列化和反序列化操作，比如在反序列化操作中对新增属性设置默认值、对类进行版本管理等。

## 45. 应该用datetime模块来处理本地时间，而不是time模块

* time模块里的行为是依赖于操作系统的，该模块的实际行为取决于底层C库函数如何与宿主操作系统交互。

* datetime模块只携带了UTC一个时区，而开源社区提供的pytz模块携带了丰富的时区信息，这样无论宿主计算机运行何种操作系统，程序的表现是一致的。

## 46. 使用内置算法与数据结构

1. 双向队列 collections.deque

这是一种双向队列，从该队列头部和尾部增删元素只需消耗常数级别的时间（时间复杂度为O(1)），非常适合做FIFO。虽然内置的list类型也可以完成同样的功能，但是从list头部增删元素的耗时是线性级别的（时间复杂度O(n)）。

2. 有序字典 collections.OrderedDict

在拥有相同键值对的dict上面迭代，可能会出现不同的迭代顺序，这是因为dict内部是使用哈希表实现的。而OrderedDict是按照键的顺序来迭代的，这种确定的行为可以简化测试和调试工作。

3. 带默认值的字典 collections.defaultdict

键入我们用内置的dict来实现计数器，总是要先查询键是否存在，不存在就初始化为0，存在就直接加1，这稍显麻烦。defaultdict可以使用int类型的默认值0作为键的默认值。

```python
stats = defaultdict(int)
stats['my_counter'] += 1
```

4. 堆队列（优先级队列） heapq

heapq模块提供了堆操作函数可以在标准的list对象上创建堆结构。

···python
a = []
heappush(a, 5)
heappush(a, 3)
heappush(a, 7)
heappush(a, 4)

# a[0]是3，数字越小表示优先级越高。
···

5. 二分查找 bisect.bisect_left

在list上使用index方法来搜索是逐个遍历的，时间复杂度是O(n)。

而bisect_left函数是二分查找的，更加高效。

```python
x = list(range(10**6))
i = bisect_left(x, 991234)
```

6. 与迭代器有关的工具 itertools

内置的itertools模块中有大量的工具可以组合和操作迭代器。echo：太多了，不写了。

## 47. 在重视精确度的场合，应该使用decimal

* python的整数类型可以表达任意长度的值，但其双精度浮点数类型遵循IEEE 754标准，是有精度损失的。

* 内置的decimal模块中有个Decimal类，使用定点数学运算，对于计算有理数是非常精确的。

* Decimal类还提供了内置函数，可以制定精度和舍入方式。
