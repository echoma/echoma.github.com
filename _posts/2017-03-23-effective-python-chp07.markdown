---
layout: post
title:  "《effective python》笔记 第7章 协作开发"
date:   2017-03-23 13:31:21 +0800
---

> Python语言的某些特性，能帮助开发者构建接口清晰、边界明确的优秀API。Python社区也形成了一套固定做法，保证程序演化过程中保持代码的可维护性。

## 49. 为每个函数、类和模块编写文档字符串

* Python是动态语言，文档就显的极为重要。Python对文档提供了内置支持，即`docstring`，我们可以通过名为`__doc__`的属性访问函数的文档：

```python
def palindrome(word):
    """This is the docstring"""
    pass

print(repr(palindrome.__doc__))


'This is the docstring'
```

* 由于文档的内置支持，产生了三方面的好处：

  1. 开发方便，可以用help函数查看文档（可以在Python解释器里做）
  2. 方便构建文档工具，如[Sphinx](http://sphinx-doc.org), [Read the Docs](https://readthedocs.org)。
  3. 社区认为文档非常重要，使得大多数开源代码都有优雅的文档。

* 更多的文档标准参阅`PEP 257`，大致总结下：

  1. 类的文档中要介绍本类的行为、重要属性、本类的子类应该实现的行为。
  2. 函数及方法的文档要介绍函数的参数、返回值、可能抛出的异常，及其他行为。

## 50. 包来安排模块，并提供稳固的API

* 代码量变大，模块数量增多时，我们会用到`包(package)`来管理代码。包提供了两大用途：

  1. 命名空间 ：使得我们在不同的包里可以编写相同文件名的模块，诸如utils.py之类。而且使我们可以使用import语句来使用相对路径引入模块。
  2. 稳固的API：通过编写`__all__`属性，可以对外部使用者暴漏严谨、稳固的API；且包不会对外暴漏以下划线开头的那些属性，只会暴漏public属性。

* 包的定义方法请读者自学，基本上就是建立`__init__.py`文件。

## 51. 为自编的模块定义根异常，以便将调用者与API相隔离

* 包里可能抛出的异常也跟包里定义的函数、类一样，是接口的重要组成部分。

* 为自己的模块定义根异常，模块内除了其他包抛出的异常外，其他所有异常都继承自该根异常。这样可以使包的使用者轻松捕获包的异常。

* 通常我们自己的包会捕获所有异常，并抛出继承自根异常的异常。如果有例外，那一定是因为包里出现了bug：

```python
# 三个层次的异常，有着不同的含义
try:
    weight = my_package.some_function()
except my_package.MySomeError:
    weight = 0 # 根据some_function()的文档，应当捕获的的已知异常
except my_package.Error as e:
    logging.error('Bug in some_function() %s', e) # 调用的函数里有异常，可能是待修复的bug
except Exception as e:
    logging.error('Bug in API: %s', e) # 未能捕获的bug，记录在日志里，包的使用者会知道这里一定是这个包有bug
    raise
```

## 52. 用适当的方式打破循环依赖关系

* 和他人协作时，可能会写出互相依赖的模块，导致程序启动式循环引入，直到崩溃。

* 解决这个问题的最佳方法，是将导致两个模块互相依赖的那部分代码重构为单独的模块，并放在依赖树的底部（最早被依赖）。

* 上面的方法的工作量可能会比较大，最简单的方案就是动态引入模块。

```python
def show():
    import app # 动态引入
    pass
```

## 53. 用虚拟环境隔离项目，并重建其依赖关系

* 不同的开发者或软件包使用了不同版本的软件包，而一个电脑只能全局安装一个版本的软件包。

* 使用pyvenv工具来解决。从Python3.4开始，该命令随着Python一起安装。而在之前的版本要通过`pip install virtualenv`命令来安装，叫做virtualenv的命令行工具。

* 使用方法自行学习。