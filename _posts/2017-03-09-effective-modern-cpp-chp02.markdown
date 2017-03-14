---
layout: post
title:  "《effective modern c++》笔记 第2章 auto"
date:   2017-03-09 13:45:21 +0800
---

# 第二章

> `auto`在概念上已经极为简化，它能节省打字时间，能避免人在正确性上的耗时。但是某些时候，`auto`的推导结果虽然忠实呈现上一章讲过的规则，但在开发者看来确实不正确的。发生这种情况时，我们应当引导`auto`产生正确的结果，而不是因噎废食回到人工输入类型声明的老路上。

> 除了前面第2-4点讲到的很多异常场景，我们还需要知道更多关于auto的信息。

## 5. 尽量用auto取代明确的类型声明

####  使用`auto`的其他好处：

* 防止忘记初始化变量：

```c++
int x; // 忘记初始化，编译通过。

auto x; // 忘记初始化，编译失败。
```

* 某些情况中，c++14的代码非常精简：

```c++
// c++11
auto compare = 
  [](const std::unique_ptr<Widget>& p1,
    const std::unique_ptr<Widget>& p2)
  { return *p1 < *p2; }

// c++14  连lambda表达式的参数都可以用auto
auto compare = 
  [](const auto& p1, const auto& p2)
  { return *p1 < *p2; }
```

* 避免类型截断(type shortcut)问题：

```c++
std::vector<int> v;
// 该代码在32位机器上是正确的，但64位机器上是有问题的，但编译是通过的
unsigned sz = v.size();
// 使用auto，避免此类问题
auto sz = v.size();
```

* 避免没有预料到的类型不符：

```c++
std::unordered_map<std::string, int> m;
// 以下代码编译失败，因为无序表的元素是std::pair<const std::string, int>
for (const std::pair<std::string, int>&p : m)
{
}
// 使用auto，避免此类问题
for (const auto& p : m)
{
}
```

#### 使用`auto`带来的其他问题：

* 影响代码可读性，变量类型变得不直观。

解决方法：`auto`并不是强制开发者使用的，很多场景是适合使用auto的。一般来说，对象是个容器、迭代器或智能指针时，我们并不需要知道具体是什么类型的容器、迭代器、智能指针，再配合适当的变量名称，代码是比较直观的。

## 6. auto推到类别与预期不符时，就是用明确的类别初始

都是废话。
