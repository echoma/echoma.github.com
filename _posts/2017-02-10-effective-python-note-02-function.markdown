---
layout: post
title:  "《effective python》笔记 第2章 函数"
date:   2017-02-10 13:28:21 +0800
---

> 本章主要讲解如何更好的书写函数、避免bug，其中使用的某些技巧是python特有的。

## 14. 尽量使用异常来表示特殊情况，而不要返回None

* 编写函数时，程序员喜欢给None这个返回值赋予特殊意义。这么做有时是合理的。

* None和0及空字符串之类的值，在条件表达式中都会被判定为False。因此，使用None做返回值表示特殊意义，很容易使调用者犯错。

* 函数遇到特殊情况应当抛出异常，调用者看到函数的文档中描述的异常之后，应当编写相应的代码来处理它们。

* echo注：这似乎规定了一种编码和文档规范，函数的文档里要写明可能会抛出的异常。

## 15. 了解如何在闭包里使用外围作用域中的变量

* 如下代码里，`helper`函数对变量`found`的赋值是无效的：

```python
def find_name(key, name_list):
    found = False
    def helper(x):
        if x in name_list:
            found = True # 这个设置是无效的
            return  (0, x)
        return (1, x)
    name_list.sort(key=helper)
    return found
```

* 应当使用`nonlocal`关键字，该关键字只会在上层作用于查找变量，而不会让变量名查询延伸到模块级别(全局作用域)：

```python
def find_name(key, name_list):
    found = False
    def helper(x):
        if x in name_list:
            nonlocal found
            found = True
            return  (0, x)
        return (1, x)
    name_list.sort(key=helper)
    return found
```
* 不幸的是，python2不支持nonlocal关键字，可以使用可变值（如包含某个元素的列表）来实现类似机制：

```python
def find_name(key, name_list):
    found = [False] # 列表本身是可修改的(mutable)
    def helper(x):
        if x in name_list:
            found[0] = True
            return  (0, x)
        return (1, x)
    name_list.sort(key=helper)
    return found
```

* echo：我觉得python2里这种代码让阅读者会觉得挺莫名其妙。

#### 16. 考虑用生成器改写返回列表的函数

* 如下函数将返回给定字符串中每个单词的起始索引：

```python
def index_words(text):
    result = []
    if test:
        result.append(0)
    for index, letter in enumerate(text):
        if letter == ' ':
            result.append(index + 1)
    return result
```

* 返回函数的列表可能会有两个问题：
    1. 函数内部要维护一个局部列表变量，随着append操作的次数增多，会占用大量内存。
    2. 函数要不停的append，最后返回列表，代码会相对使用生成器啰嗦。（echo：我觉得还好）

* 用生成器代码改写：

```python
def index_words_iter(text):
    if test:
        yield 0
    for index, letter in enumerate(text):
        if letter == ' ':
            yield index + 1

# 调用方法：
result = list(index_words_iter(address))
```

* echo：跟第9条类似，生成器可以减少内存占用。