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

## 16. 考虑用生成器改写返回列表的函数

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

## 17. 在参数上迭代时，要多加小心

* 假设我们写了如下函数，该函数接会把输入的数值加总，然后给出每个数字占的百分比：

```python
def foo(numbers):
    total = sum(numbers)
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result
```

* 给上面的函数传入一个列表，是可以得到正确结果的。但如果给上面函数传入一个迭代器只能得到空的结果。因为迭代器只能遍历一次，而函数遍历了两次`numbers`参数（一次求和，一次for循环）。进行第二遍迭代也不会抛出任何异常，因为for循环认为迭代过程中出现迭代到终点的很正常的。

* 改造思路一：我们可以使用`list(numbers)`拷贝一份列表，在列表上进行求和、for循环，但这可能会导致大的内存占用。

* 改造思路二：也可以改变foo函数参数，传入一个生成器函数，我们通过生成器获得两个迭代器对象，一个用于求和，一个用于for循环。

* 改造思路三：也可以限制用户输入的`numbers`参数必须是容器，而不能是迭代器：

```python
# iter函数有如下约定：
#    如果把迭代器对象传给iter函数，则返回该迭代器；
#    如果传入容器，则生成一个新的迭代器并返回。
if iter(numbers) is iter(numbers):
    raise TypeError('Must supply a container')
```

## 18. 用数量可变的未知参数减少视觉杂讯

* 星号参数(`*args`或`star args`)，能够使代码更加清晰，减少视觉上的干扰。考虑如下函数：

```python
def log(message, values)
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print('%s: %s' % (message, values_str))

log('My numbers are', [1, 2])
log('My numbers are', [])
```

* 可以用星号参数改写如下：

```python
def log(message, *values) # values前面加个*
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print('%s: %s' % (message, values_str))

log('My numbers are', 1, 2) # 调用者就像传输普通位置参数一样随便传多少个

# 也可以传入生成器
def my_generator():
    for i in range(10):
        yield i
it = my_generator()
log('My numbers are', *it)
```

## 19. 用关键字参数来表达可选的行为

* 考虑如下函数，它的最后两个参数可以指定忽略ZeroDivisionError和OverflowError错误：

```python
def safe_division(number, divisor, ignore_overflow, 
                  ignore_zero_division):
    pass
```

* 调用者有时只想忽略ZeroDivisionError，有时只想忽略OverflowError，就会写出这样的代码：

```python
safe_division(a, b, True, False)
safe_division(a, b, False, True)
```

* 函数改用可选参数，调用者使用关键字形式来调用，代码会更加清晰，调用意图更明确：

```python
def safe_division(number, divisor, ignore_overflow=False, 
                  ignore_zero_division=False):
    pass

# 调用
safe_division(a, b, ignore_overflow=True)
safe_division(a, b, ignore_zero_division=True)
```

* 对于可选的关键字参数，应当总是以关键字形式来指定，不应以位置参数的形式指定。这样代码更清晰。

## 20. 用None和文档字符串来描述具有动态默认值的参数

* 考虑如下函数：

```python
def log(message, when=datetime.now()):
    print('%s: %s' % (when, message))

# 调用两次
log('a')
log('b')
```

* 上面的函数连续调用两次输出的时间是一样的，因为参数when只会被初始化一次（模块被加载是求出）。

* 如果向实现动态默认值，习惯上是把默认值设为None，并在文档字符串(docstring)中把None所对应的行为描述出来：

```python
def log(message, when=None)
    """
    Args:
        when: date time of when the message occurred.
            Defaults to the present time.
    """
    pass
```

* 如果可选参数的默认值是`{}`或`[]`等动态值，多次调用时其实会共享同一个字典或列表对象，导致奇怪的行为。这种情况同样遵循上面的规则，使用None来做默认值。

## 21. 用只能以关键字形式指定的参数来确保代码清晰

* 为了保证调用者必须已关键字形式给出可选参数，我们可以在参数列表里加入`*`号，表示位置参数在此结束，之后的参数只能以关键字形式指定：

```python
def safe_division(number, divisor, * ignore_overflow=False, 
                  ignore_zero_division=False):
    pass

# 调用时使用位置参数指定后两个参数会导致异常
safe_division(a, b, True False)
>>>
TypeError: safe_division() takes 2 positional arguments but 4 where given
```

* 不幸的是，python2不支持这个语法，但我们可以用`**args`来模拟。`**args`可接收任意个数的参数，我们通过`pop`掉我们需要的参数后，如果`**args`还不为空则表示客户没有传入指定的关键字：

```python
def safe_division(number, divisor, **kwargs):
    ignore_overflow = kwargs.pop('ignore_overflow', False)
    ignore_zero_div = kwargs.pop('ignore_zero_division', False)
    if kwargs:
        rase TypeError('Unexpected **kwargs: %r' % kwargs)
```