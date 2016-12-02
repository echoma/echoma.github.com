---
layout: post
title:  "Linux主机共享网络给Windows系统上网"
date:   2015-07-01 20:28:21 +0800
categories: linux network 折腾
---

## 需求背景

* 我的办公电脑是一台使用`Ubuntu Linux系统`的台式机。

* 我带来了自己的`Windows系统`的笔记本电脑到公司里用，但是不想用Wifi上网（太慢）。

* 我的办公卡座下就只有一个网线插口，已经被办公电脑用了。

* 这台办公电脑只有一块网卡，用网线插在了卡座唯一的插口上，用来上网。

那么问题来了，怎么才能让这台笔记本电脑通过这台Linux主机上网呢？

## 解决方案的大概思路

三步走：

1. Linux主机上加装一个USB有线网卡，让Windows主机通过网线连接到Linux主机。

2. Linux系统里对这个USB网卡包转发到原来上网的网卡上。

3. Windows系统把网关设置为Linux主机，就可以上网啦。

## 实施步骤一：让Windows主机有线连接到Linux主机

淘宝上搜索“USB有线网卡”，更准确点的是搜“AX88772C 网卡”。这是个一头是usb，一头是RJ45网口的小东东。AX88772C是这种网卡使用的芯片的型号。通常二三十块钱，我买的绿联的。

准备一条交叉线，淘宝上一样可以买到，长度根据你两台电脑之间的距离自己选择。我买了2米的。

把USB网卡插到Linux主机的USB接口上，安装驱动：

* 首先要去找网卡驱动。有的买来的网卡就带了驱动光盘，也可以找找有没有Linux驱动。这种网卡的芯片都是AX88772C的，google一下就可以下载到Linux的驱动源码。

* 源码解压后是类似这样的文件夹：`AX88772C_772B_772A_760_772_178_LINUX_DRIVER_v4.17.0_Source`

* 根据文件夹里README文件来编译就可以了，基本上就是 `make && make install`

* 如果编译碰到类型datetime的报警，就需要修改下makefile里的`EXTRA_CFLAGS`，给它加个`no-error`选项，如这样：`EXTRA_CFLAGS = -DEXPORT_SYMTAB -Wno-error=date-time`

* 编译、安装完成后，还需要按照README里写的启动一下这个驱动模块：`modprobe asix`

网卡启动后，系统会在eth0后面多出个eth1，设置下IP，比如：`192.168.0.10 （子网掩码 255.255.255.0  网关 192.168.0.1）`

然后在eth1的路由选项里，勾上`仅将此链接用于相应的网络上的资源`以及`忽略自动获取的路由`（如果这个选项可以勾就勾上）

## 实施步骤二：Linux系统设置端口转发，共享网络

切换到root账户，执行以下命令：

```
echo "1" > /proc/sys/net/ipv4/ip_forward
iptables -F
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

注意：上面这行里的eth0就是Linux机器上能上网的那个网络设备。

## 实施步骤三：Windows上设置网关为Linux系统

windows上用交叉线连上linux那个网卡后，系统会多出个连接，win7的话在弹出对话框里选择“办公网络”。

给它设置ip：`192.168.0.20 （子网掩码 255.255.255.0  网关 192.168.0.10）`，注意网关就是Linux那边eth1的ip

把DNS设置为`4.4.4.4`，或者你公司内部的DNS服务器，不然域名是解析不了的。

这时在Windows上`ping 192.168.0.10`是可以通的。

然后在控制面板里，将这条新链接的防火墙关闭。

这时去Linux上`ping 192.168.0.20`也是通的了。

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

## 共享文件、共享鼠标键盘

这时Linux和Windows其实是在一个小的子网里了，所以不仅能共享网络上网，共享文件也是水到渠成的事，装个`samba`就可以了。

共享鼠标键盘就用`synergy`就可以了。

`samba`和`synergy`的使用方法这里不做介绍。