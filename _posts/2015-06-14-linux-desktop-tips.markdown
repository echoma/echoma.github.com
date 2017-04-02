---
layout: post
title:  "常年使用linux桌面积累的小tips"
date:   2015-06-14 20:28:21 +0800
---

* 目录
{:toc}

## 修改ubuntu的默认文本编辑器

sudo gedit /etc/gnome/defaults.list

然后批量替换里面的gedit.desktop为sublime_text.desktop 

## sublime_text3可以输入中文

参看这篇文章：http://blog.csdn.net/cywosp/article/details/32350899

注意几点：1. 输入法必须是fctix，比如我们用搜狗拼音。2.原文里的gcc命令里的单引号应该是`号。 

## 修改主机名称

1.执行：hostname oratest 

2.修改/etc/sysconfig/network中的hostname 

3.修改/etc/hosts文件 

## 修改IP配置

vi /etc/sysconfig/network-scripts/ifcfg-eth0

里面的ONBOOT=yes表示开机就启用此设备，BOOTPROT是dhcp表示动态获取，static表示静态IP。

如果配置了static，那要增加IPADDR=192.168.0.41和GATEWAY=192.168.0.1

配置完成后，保存，然后重启网络(service network restart)

以上是要重启才生效的，如果想要立刻生效可以调用命令：

```bash
ifconfig eth0 192.168.0.2 netmask 255.255.255.0 
route add default gw 192.168.0.1 dev eth0
```

注意：如果从virtualbox复制了虚拟机要先修改mac地址，如果还是启动不了eth0，那要找 /etc/udev/rule.d/70-persistent-net.rules，把这个文件里旧的mac地址的记录删掉，或者把这个文件删掉也可以，最后reboot。

注意：新的机器可能没有dns，可以先配成dhcp获取下dns，再配置成static。或者直接编辑/etc/resolv.conf写入以下内容：

```ini 
nameserver 156.154.70.22 
nameserver 8.8.8.8 
nameserver 202.96.128.166 
nameserver 202.96.134.133 
```

## VirtualBox技巧

1. XP安裝後CPU100%的解决办法如下： 
    在“我的电脑” 上单击右键,选择“硬件--设备管理器”，在设备管理器中选择“计算机”展开，选择“ACPI Multiprocessor PC” ，在该项上点击右键，选择“更新驱动程序”，选择“从列表或置顶位置安装”,下一步，选择“不要搜索，我要自己选择安装的驱动程序”，下一步，选择“Standard PC”,接着下一步，完成安装。 

2. 改变已存在磁盘的uuid： VBoxManage internalcommands sethduuid ./winxp2.vdi  这样复制磁盘文件直接使用是不会报“UUID已存在”的错误。 

## Ubuntu11.10下Eclipse CDT 代码悬浮提示窗口背景黑色设置方法

在eclipse里直接可以设置，修改该颜色的配置选项位于：`菜单栏 Window->preferences->C/C++->Editor 项目中的Appearance color options里面的Source hover background选项`，取消勾选System Default，选择喜欢的颜色即可。 

## Eclipse在kernel升级后，CDT提示项目有invalid include path的解决方法

关闭eclipse，然后工作空间下这个目录/data/work/usfsi/.metadata/.plugins/org.eclipse.cdt.make.core里有很多sc文件，直接move到一个备份文件夹，重启eclipse就可以了。 

## thunderbird邮件配置

打开：编辑-》首选项-》高级-》常规-》配置编辑器

设置回复邮件时“新的内容在上边”：把mail.identity.default.reply_on_top设置为true。

防止大附件解压：mail.inline_attachments设置为false。

防止CPU老是很高：mail.db.idle_limit 从30万调到3000万 

## 添加系统自动启动

ubuntu的lens里搜索“启动”出现的窗口里就编辑启动项了。也可在这个目录  ~/.config/autostart/  下面添加快捷方式。 

## EPEL

网址： http://fedoraproject.org/wiki/EPEL 

命令： rpm -ivh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-7.noarch.rpm 

## guake能自己修改宽度和高度的

1.备份先

    代码：sudo cp /usr/lib/guake/guake.py ~/guake

2.用默认文本编辑器打开

    代码：gksu leafpad /usr/lib/guake/guake.py

3.查找width = 100

    修改成自己想要的宽度，比如80

4.查找window_rect.y = 0

    修改成30，guake就不再挡住上面的面板了 

## linux上连接ssh服務器很慢的解決辦法

修改本机的客户端配置文件ssh_conf，注意，不是sshd_conf

找到 `GSSAPIAuthentication yes` 改为 `GSSAPIAuthentication no`

## XFCE设置caps lock作为ctrl： 

sudo mousepad /etc/default/keyboard

把option改成：  XKBOPTIONS="ctrl:nocaps" 

## wunderlist在ubuntu上的安装：

来源：https://support.wunderlist.com/customer/portal/questions/9457896-ubuntu-14-4

1. Install Chrome Browser 

2. Go in Chrome Web Store and search for Wunderlist and install it 
  *Chrome Web Store link: https://chrome.google.com/webstore/category/apps 

3. After installing open new tab in Chrome and type "chrome://apps/" without quotes in chrome address bar, or just click the "Apps" icon at the left top corner if available 

4. You should see Wunderlist launch icon among all other Chrome Apps 

5. Launch and when you see Wunderlist icon in your launcher at the left side, right click to it and choose "Lock to Launcher" 

6. Now you have your launch icon for Wunderlist on your desktop :) 

7. Enjoy it

## 重启搜狗拼音输入法：

```bash
#!/bin/sh
pidof fcitx | xargs kill
pidof sogou-qimpanel | xargs kill
nohup fcitx  1>/dev/null 2>/dev/null &
nohup sogou-qimpanel  1>/dev/null 2>/dev/null &
```