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