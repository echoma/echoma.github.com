---
layout: post
title:  "《effective python》笔记"
date:   2017-01-16 20:28:21 +0800
---

## pythonic方式

#### 1. python版本

* python2功能开发已经冻结，只做bug修复、安全增强和移植等工作。
* python社区的开发重点是python3，开发后续项目应当首选python3。
* 2to3与[six](https://pythonhosted.org/six/)可以帮助大家把代码轻松的是配到python3及后续版本。

#### 2. 遵循PEP8

* 没啥，就是总结了主要的风格特征

#### 3. 了解bytes\str\unicode

* python3：`bytes`是二进制，`str`是unicode字符。
* python2: `str`是二进制，`unicode`是unicode字符。
* 程序的核心逻辑尽量使用unicode字符。
* 不要对输入的字符编码做任何假设。可以编写helper函数方便把任何参数转为unicode。

#### 4. 用辅助函数取代复杂晦涩的表达式

* python语法太精炼，开发者容易过度使用语法特性，写出晦涩的单行长表达式。
* 最好把长表达式移入函数中。
* 尽量使用新的三元表达式，它更清晰。

#### 5. 了解序列切片(slice)

* 不写多余代码，索引留空会更清晰（ `a[:10]` 比 `a[0:10]` 清晰）
* 切片操作不用担心索引越界（可以用来限定输入的数组的最大尺寸，`a = input[:20]`就限定了最大20个元素）
* 赋值操作中，切片位于`=`的左侧和右侧是有不同表现的：
  - 左侧`b = a[:]`：会产生a的拷贝，然后赋值给b。b和a的内容相同，但不是同一份引用。
  - 右侧`b = a  \n  a[:] = [1,2,3]`：b和a仍然是同一份引用。

#### 6. 单次切片操作内，不要同时指定start\end\stride

* `list[start:end:stride]`：表示切片时每stride个元素中去一个出来。
* 负数的`stride`切片在utf-8等编码的unicode字符可能会引发异常，代码中最好不要出现负数`stride`。
* `list[start:end:stride]`这种代码通常导致难以阅读，建议拆分为`list[start:end]`+`list[::stride]`两次操作。
* 如果无法按上述建议拆分，可以考虑内置itertools模块的islice方法。