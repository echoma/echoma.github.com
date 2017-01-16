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

* python3：bytes是二进制，str是unicode字符。
* python2: str是二进制，unicode是unicode字符。
* 程序的核心逻辑尽量使用unicode字符。
* 不要对输入的字符编码做任何假设。可以编写helper函数方便把任何参数转为unicode。

#### 4. 用辅助函数取代复杂晦涩的表达式

* python语法太精炼，开发者容易过度使用语法特性，写出晦涩的单行长表达式。
* 最好把长表达式移入函数中。
* 尽量使用新的三元表达式，它更清晰。
