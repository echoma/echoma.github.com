---
layout: post
title:  "《effective python》笔记 第6章 内置模块"
date:   2017-03-09 16:47:21 +0800
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