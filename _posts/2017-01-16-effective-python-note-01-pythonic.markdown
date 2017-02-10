---
layout: post
title:  "《effective python》笔记 第1章 用pythonic方式思考"
date:   2017-01-16 20:28:21 +0800
---

> pythonic：一门语言的编程习惯是由用户确立的，python开发者用pythonic来描述符合特定风格的代码。
>
> 本章主要是讲解如何使用python特有的方式来书写直观、简洁、易读的代码。

## 1. python版本

* python2功能开发已经冻结，只做bug修复、安全增强和移植等工作。
* python社区的开发重点是python3，开发后续项目应当首选python3。
* 2to3与[six](https://pythonhosted.org/six/)可以帮助大家把代码轻松的是配到python3及后续版本。

## 2. 遵循PEP8

* 没啥，就是总结了主要的风格特征

## 3. 了解bytes\str\unicode

* python3：`bytes`是二进制，`str`是unicode字符。
* python2: `str`是二进制，`unicode`是unicode字符。
* 程序的核心逻辑尽量使用unicode字符。
* 不要对输入的字符编码做任何假设。可以编写helper函数方便把任何参数转为unicode。

## 4. 用辅助函数取代复杂晦涩的表达式

* python语法太精炼，开发者容易过度使用语法特性，写出晦涩的单行长表达式。
* 最好把长表达式移入函数中。
* 尽量使用新的三元表达式，它更清晰。

## 5. 了解序列切片(slice)

* 不写多余代码，索引留空会更清晰（ `a[:10]` 比 `a[0:10]` 清晰）
* 切片操作不用担心索引越界（可以用来限定输入的数组的最大尺寸，`a = input[:20]`就限定了最大20个元素）
* 赋值操作中，切片位于`=`的左侧和右侧是有不同表现的：
  - 左侧`b = a[:]`：会产生a的拷贝，然后赋值给b。b和a的内容相同，但不是同一份引用。
  - 右侧`b = a  \n  a[:] = [1,2,3]`：b和a仍然是同一份引用。

## 6. 单次切片操作内，不要同时指定start\end\stride

* `list[start:end:stride]`：表示切片时每stride个元素中取一个出来。
* 负数的`stride`切片在utf-8等编码的unicode字符可能会引发异常，代码中最好不要出现负数`stride`。
* `list[start:end:stride]`这种代码通常导致阅读困难，建议拆分为`list[start:end]`+`list[::stride]`两次操作。
* 如果无法按上述建议拆分，可以考虑内置itertools模块的islice方法。

## 7. 用列表推导来取代map和filter

* 就是介绍了下列表推导。
* echo注：用pylint就会提示map和filter已经不建议使用了。

## 8. 不要使用含有两个以上表达式的列表推导

* 列表推导支持多重循环。例如：

```python
matrix = [[1,2,3],[4,5,6],[7,8,9]]
flat = [x for row in matrix for x in row]
print(flat)

>>>
[1,2,3,4,5,6,7,8,9]
```

* 含有超过两个(即3个及3个以上)循环的列表推导，很难理解，还不如直接用for循环。for循环有缩进，代码会更清晰。如下：

```python
# 三重列表推导
flat = [x for sublist 1 in my_lists
        for sublist2 in sublist 1
        for x in sublist2]

# 改写为三重for循环
flat = []
for sublist1 in my_lists:
    for sublist2 in sublist1:
        flag.extend(sublist2)
```

## 9. 用生成器表达式来改写数据量较大的列表推导

* 列表推导的缺点：在推导过程中，对于输入序列的每个值，可能都要创建仅含一项元素的全新列表。如果输入序列很大，可能会消耗大量内存。例如：

```python
# 读取文本文件，并返回没一行的长度
value = [len(x) forx in open('/tmp/my_file.txt')]
print(value)

>>>
[100,57,15,1,12,75,5,86,89,11]
# 内存里会缓存所有行的长度，如果文件很大，或者是通过无休止的网络来读取的，那会占用大量内存。
```

* 为了解决此问题，python提供`生成器表达式(generator expression)`，他是对列表推导和生成器的泛化。核心是`迭代器`，每次根据生成器只产生一项数据。例如：

```python
it = (len(x) for x in open('/tmp/my_file.txt'))
print(it)

>>>
<generator object<genexpr> at 0x101b81480>

print(next(it))
print(next(it))

>>>
100
57

# 不需要担心内存激增
```

* 生成器表达式的另一个优点，是可以相互组合。例如：

```python
# 把一个生成器表达式返回的迭代器作为另一个生成器表达式的输入
roots = ((x, x**0.5) for x in it)
print(next(roots))

>>>
(15, 3.872983346207417)
```

* 生成器表达式的问题是，迭代器是有状态的，用过一轮就不能反复使用了。

## 10. 尽量用enumerate取代range

* 在遍历列表时，如果要知道元素的索引，通常会使用range：

```python
for i in range(len(item_list)):
    item = item_list[i]
    print('%d: %s' % (i + 1, item))
# 这种代码有些生硬，不易理解
```

* 使用内置的enumerate函数，解决此问题：

```python
for i, item in enumerate(item_list):
    print('%d: %s' % (i + 1, item))
```

* enumerate函数还可以指定开始计数时使用的值：

```python
for i, item in enumerate(item_list, 1):
    print('%d: %s' % (i, item))
# 本例从1开始计数，代码更简短
```

## 11. 用zip函数同时遍历两个迭代器

* 内置的zip函数可以将多个迭代器封装为生成器，以便稍后求值。例如：

```python
for name,info in zip(names, infos):
    print('%s: %s' % (name, info))
```

* 注意，python2的zip不是生成器，而是遍历列表后直接生成所有元素组合成的列表，这可能会占用较大内存。

* zip函数遍历时，如果多个迭代器长度不等，zip函数以短的那个为准。

## 12. 不要在for和while循环后面写else块

* python提供了其他语言都不支持的特性：

```python
for i in range(3):
    print('%d' % i)
else:
    print('else block')
```

* 通常程序员会理解为：如果循环没有正常执行完，就执行else块。但实际刚好相反，如果循环里使用break提前跳出，就不执行else块。换句话说，如果循环正常执行完毕，才执行else块。

* 这种语法不直观，容易让人误解，不要使用。

## 13. 合理利用try/except/else/finally中的代码块

* 每个块都有特殊用途，可以有多种搭配：

```python
try:
    # 可能会抛出异常的代码
except ValueError as e:
    # 异常处理代码
else:
    # try块顺利执行后，会执行这段代码
finally:
    # 无论异常是否发生，都要做的清理工作
```

* finally块通常用来做清理工作。
* else块可以缩减try块的内容，将无异常发生时的代码与try/except代码分开。