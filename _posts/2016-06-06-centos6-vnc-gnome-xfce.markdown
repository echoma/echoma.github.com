---
layout: post
title:  "通过VNC连接在CentOS6的远程桌面"
date:   2016-06-06 20:28:21 +0800
---

## 需求背景

* 之前一直在自己的办公机上写代码，是`Ubuntu16.04+XFCE4+Eclipse`

* 但是自己的办公机配置比较老，`Eclipse`这个玩意是java的，项目大了就变得卡了，即使使用SSD也没用

* 正好公司的开发机要迁移，都是重新整过的新机器新环境，配置都非常好，`Eclipse`在那上面应该会跑的非常流畅。

* 开发机的系统都是没有安装桌面环境的，要自己动手装

那么问题来了，怎么如果配置远程桌面，让我的Ubuntu远程连到服务器上做开发呢。

## 解决思路

三步走：

1. 在远端机上（我这里是CentOS6)上安装桌面环境。

2. 在远端机上配置VNC服务器。

3. 在本地机（我这里是Ubuntu16.04)上使用VNC客户端登陆到远端。

## 实施步骤一：在远端机上上安装桌面环境。

#### 1.安装桌面环境软件

折腾的过程中发现，如果要想使用XFCE桌面环境，必须也得把GNome桌面环境装好。

所以，第一步要安装GNome，使用root执行：`yum groupinstall -y basic-desktop desktop-platform x11 fonts`

#### 2.修改系统的启动级别为X11

我们的开发机默认是命令行多用户模式启动，我们需要切换到X11图形模式，使用root执行`vi /etc/inittab`，将里面的`id:3:initdefault:`替换为`id:5:initdefault:`

#### 3.重启机器，检察GNome是否正常启动

用root执行reboot命令进行重启，重启后使用ps命令看GNome的相关进程是否存在：`ps afxu | grep gnome`

通常会有如下进程

```bash
gnome-settings-daemon
gnome-screensaver
gnome-volume-control-applet
```

## 实施步骤二：在远端机上安装远程桌面服务器

#### 1. 安装VNC服务

使用root执行：`yum install -y vnc-server`

#### 2. 确保VNC服务开机自启动

使用root执行：`chkconfig vncserver on`

#### 3.创建用户

假如你打算就用现有的账户登录桌面，那就可以跳过这步了。

以创建用户xxx为例，使用root执行：

```bash
useradd xxx
passwd xxx
```

#### 4.配置用户可以使用vnc

  **a.** 使用root修改vnc配置文件，为新用户增加vnc配置：`vi /etc/sysconfig/vncservers`

示例配置：

```ini
VNCSERVERS="1:echo 2:xxx"
VNCSERVERARGS[1]="-geometry 1024x768"
VNCSERVERARGS[2]="-geometry 1900x1050"
```

上面的`xxx`就是上一步创建的用户，`1024x768`就是分辨率。

  **b.** 接上一步，执行以下命令可以切换到新用户并创建vnc密码：

```bash
su - xxx    #切到xxx用户
vncpasswd   #设置vnc登录密码
```

  **c.** 然后切回root，重启vnc服务，检查过程中是否有任何报错，这个过程vnc服务也会在该xxx用户名下创建初始化脚本(`/home/xxx/.vnc/xstartup`)：

执行以下命令：
```bash
exit    #切回root
service vncserver stop
service vncserver start
```

*常见的问题*：在启动过程中报类似`invalid domain name somename.localdomain:1 in 'add' command`的问题，只要把这个域名（不含后面的:1）配到`/etc/hosts`里就可以解析了。示例`/etc/hosts`:

```
127.0.0.1  localhost  localhost.localdomain  somename.localdomain
172.18.8.120 gitlab.futunn.com
```

  **d.** 修改用户的xstartup文件，确保桌面环境使用了gnome（而不是默认的twm)

通常默认的xstartup是这样的：

```bash
#!/bin/sh

[ -r /etc/sysconfig/i18n ] && . /etc/sysconfig/i18n
export LANG
export SYSFONT
vncconfig -iconic &
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
OS=`uname -s`
if [ $OS = 'Linux' ]; then
  case "$WINDOWMANAGER" in
    *gnome*)
      if [ -e /etc/SuSE-release ]; then
        PATH=$PATH:/opt/gnome/bin
        export PATH
      fi
      ;;
  esac
fi
if [ -x /etc/X11/xinit/xinitrc ]; then
  exec /etc/X11/xinit/xinitrc
fi
if [ -f /etc/X11/xinit/xinitrc ]; then
  exec sh /etc/X11/xinit/xinitrc
fi
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
xterm -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
twm &
```

我们需要注释掉最后两行，并在最后增加这样一行：`exec gnome-session`

注意：不要注释掉第6行`vncconfig`命令，这个程序不是配置程序，而是它本身就提供了同步剪贴板，所以要一直开着。

## 实施步骤三：使用VNC客户端连接远程桌面

假设我们的远端机器的IP是192.169.1.20

#### 常见的linux平台的vnc客户端使用方法

  **a.** 最简易的客户端`gvncviewer`

使用如下命令：`gvncviewer 192.169.1.20:2`

上面命令里的`192.169.1.20`是服务器IP，`:2`表示使用上文vncservers配置文件里的用户2（也就是xxx这个用户)

这时会出来一个小对话框，这时输入ssh密码，然后会出现图形界面并要求输入vnc密码。

简易在vncviewer的菜单里选择全屏(full screen)，并去掉拉伸显示(scaled display)

  **b.** 最好用的客户端`krdc`

这是kde桌面套件里的一员，kde套件向来是功能最强工作最稳定的。该客户端支持vnc和rdp，可以同步剪切板，可以管理连接，工作非常稳定。

  **c.** 还不错的客户端`remmina`

remmina在启动后会在右下角有个系统托盘。功能类似krdc。但是剪切板不太稳定，偶尔还会卡死。

可以使用`remmina -i &`来启动remmina的系统托盘。

#### 必读：使用中的问题解决

  **a.** 总是弹出一个对话框要求输入root密码，写着类似`Authentication is required to set the network proxy used for downloading packages`的文字。解决方法如下：使用root编辑`/etc/xdg/autostart/gpk-update-icon.desktop`文件，然后在末尾增加一行：`X-GNOME-Autostart-enabled=false`

  **b.** 将某个用户的默认语言切换为中文：`vim ~/.vnc/xstartup`，将`export LANG`替换为`export LANG=“zh_CN.UTF-8”`，然后退出并重新登录就切换为中文环境了。

  **c.** 总是会有个vncconfig的小窗口，将上面提到的xstartup脚本里的`vncconfig -iconic &`替换为`vncconfig -nowin &`就可以了

## 使用XFCE4

1. 在安装了gnome的前提下，安装xfce软件包：`yum groupinstall xfce`

2. 将某个用户的登录环境切换为xfce，`vim ~/.vnc/xstartup`，做如下修改：
	a. 将`exec gnome-session`替换为`startxfce4 &`
    b. 将带有xinitrc的代码行注释掉：

```bash
#if [ -x /etc/X11/xinit/xinitrc ]; then
#  exec /etc/X11/xinit/xinitrc
#fi
#if [ -f /etc/X11/xinit/xinitrc ]; then
#  exec sh /etc/X11/xinit/xinitrc
#fi
```

3. p键和tab键无法输入：这是由于默认的快捷键设置跟vnc不兼容，可以修改`/home/【用户名】/.config/xfce4/xfconf/xfce-perchannel-xml/xfce4-keyboard-shortcuts.xml`快捷键文件，搜索`Super`，将跟Super键相关的行全部注释，xml的注释是在内容功能前加`<!--`，在内容后加`-->`，如下第2行：

```xml
<property name="&lt;Alt&gt;F8" type="string" value="resize_window_key"/>
<!--<property name="&lt;Super&gt;Tab" type="string" value="switch_window_key"/>-->
<property name="Escape" type="string" value="cancel_key"/>
```
4. ctrl+F2键跟eclipse快捷键冲突：ctrl+F2是旧版xfce4里切换到第二个工作区的快捷键，eclipse里是停止调试的快捷键，冲突了。同样也是修改快捷键文件（如上一条），搜索`workspace`，将类似`<property name="&lt;Control&gt;F4" type="string" value="workspace_4_key"/>`的行注释掉就可以了，从F1到F11共有11行。

## 图形性能

#### 抗锯齿和点阵字体

为了提高图形性能，减少对远程机器的CPU和内存占用，我们会选择关闭抗锯齿【系统设置->外观(Apperance)中关闭】，但这会导致矢量字体出现大量锯齿，因此我们会选用点阵字体。

对于整个桌面环境，可以在系统设置->窗口管理器或外观里设置系统默认使用自带的“文泉驿点阵”字体。

对于编程，通常是使用等宽字体，以下是几个比较好的开源等宽字体：

  1. terminus（推荐）: 该字体对常用的字号都有对应的点阵字体，非常清晰。下载：wget -O terminus-font-4.40.tar.gz http://downloads.sourceforge.net/project/terminus-font/terminus-font-4.40/terminus-font-4.40.tar.gz?r=http%3A%2F%2Fterminus-font.sourceforge.net%2F&ts=1465537121&use_mirror=heanet

  2. Bitstream Vera Sans Mono：http://www.gnome.org/fonts/ or http://ftp.gnome.org/pub/GNOME/sources/ttf-bitstream-vera/1.10/ttf-bitstream-vera-1.10.tar.gz