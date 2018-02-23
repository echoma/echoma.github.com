---
layout: post
title:  "《effective modern c++》笔记 第8章 调教"
date:   2017-08-04 13:26:21 +0800
---

* 目录
{:toc}

# 第八章

* 绝大多数情况下，我们都能很清晰的说明何种场景适合使用何种C++技巧。本章要介绍两个适用时机比较复杂的场景。

## 41. 对于低移动成本且总是需要拷贝的可拷贝参数，考虑使用传值

* 在C++98时代，我们通常会使用`const引用`来避免拷贝，以提高性能：

```c++
class Widget {
  public:
    void addName(const std::string& name); // 使用const引用避免拷贝
    {
    	this->names.push_back(name); // 在本例中，放入容器，会发生一次拷贝
    }
};
```

* 在C++11时代，我们还想增加对右值引用参数的支持，以提供更好的性能。于是上面的类变成：

```c++
class Widget {
  public:
    void addName(const std::string& name); // 使用const引用避免拷贝
    {
    	this->names.push_back(name); // 在本例中，放入容器，会发生一次拷贝
    }

    void addName(std::string&& name); // 接受右值引用
    {
    	this->names.push_back(std::move(name)); // 避免了一次拷贝
    }
};
```

* 但这样的代码会比较麻烦：
  1. 我们要维护两个版本的addName，其实代码逻辑是一样的，只是参数的处理逻辑不相同。
  2. 因为有两个函数，编译后最终生成的程序尺寸会变大。

* 当然，我们前面讲的完美转发是可以解决这个问题的，但是完美转发会带来一些技术复杂度，而且需要我们把函数设计为模板形式，导致函数的实现只能放在头文件里。

* 在上面这个场景里，我们可以考虑直接使用`传值`做参数。

```c++
class Widget {
  public:
    void addName(std::string name); // 使用传值参数，会发生拷贝
    {
    	this->names.push_back(std::move(name)); // 使用移动操作，不会拷贝。
    }
};
```

* 上面的这种写法，比起完美转发要简单很多，成本也只有一次拷贝，跟C++98时代的版本成本一样。而且他还可以接收右值引用，成本也是一次拷贝。

* 但我们必须强调，这种写法只适用于类似例子里的这种场景，并不是适用于所有场景。场景的要求是：
  1. 参数一定会被拷贝。否则用上面这种写法，只会增加成本。当然，这个参数也必须是可复制的。
  2. 移动成本很低。移动成本如果很高，那就只保留C++98时代的写法就可以了。

## 42. 考虑定位而非插入

* 介绍了[vector::emplace_back](http://en.cppreference.com/w/cpp/container/vector/emplace_back)，需要了解标准库里提供了的这种新接口是支持完美转发的，对于传参时使用了临时变量的情况性能会更好。

```c++
std::vecotr<std::string> vs;
vs.push_back("abc"); // 构造一个临时对象时会复制一次，vector还会使用移动构造函数再复制一次。
vs.emplace_back("abc"); // 就只有vector的一次构造函数。
```

* 其他内容都不太重要。