---
layout: post
title:  "积累的linux命令号小tips"
date:   2016-07-05 20:28:21 +0800
---

## 查看端口号被哪个进程占用/查看进程占用了哪些端口

lsof -i:9000 

lsof -i -n -P | grep [pname|pid]

## tcpdump使用技巧

tcpdump -Xnlps0 udp[20:2]=0x5555

tcpdump -ilo  -X port 7400 and \(tcp[40:2]==0x183a\)

tcpdump -ieth1 src host 172.30.18.182 and port 7400 and \(tcp[40:2]==0x189d\)

抓住tcp的80端口上尺寸小于1000字节且不是ACK的数据包: `tcpdump -ieth0 -X 'tcp and port 80 and ip[2:2] < 1000 and tcp[13]!=16'`   

更多技巧看这里http://wenku.baidu.com/view/fc8342ddce2f0066f5332283.html 

## 正则表达式find统计代码行数

find ./ -regex ".*\.\(h\|c\|cpp\)$" | xargs wc -l 

## 启动ntop

ntop -i em1 -M -w 9528 -d 

## 查看進程的線程

ps -aml  數量和每個線程當前系統調用

ps -o thcount -p <process id>  數量

ps -o pid,comm,user,wchan,thcount -p <process id>  數量、主進程當前系統調用

ps -eo pid,comm,user,wchan,thcount -p <process id>  比上面多了PSR（所在CPU号）

ps xH | grep <pid>         

## 查找工程目錄下的編譯好的二進制服務和庫，然後刪除

```bash
find ./ -regex ".*fsi_svr_.*\/.*_svr" -type f  | xargs rm -rf 
find ./ -regex ".*fsi_svr_.*\/.+\.a" -type f  | xargs rm -rf 
find ./ -name "*.o" | xargs rm -rf 
find ./ -name "*.d" | xargs rm -rf 
find ./ -name "*.log" | xargs rm -rf
```

## VI里批量替换

冒号后面输入`%s/172.30.18.21/172.30.18.27/g`就把ip换掉了 

## 命令行sed批量替换多个文件中的指定内容

方法1 : `find ./ -name "testsed.txt" | xargs sed -i "s/test/hhhh/g"`

方法2 ： `sed -i "s/mahuinan/huinanma/g" 'grep mahuinan -rl /www'` 

## 将某个逗号分隔的号码文件去重排序

```
    cat test.txt | awk -F ',' '{for(i=1;i<=NF;i++) print $i}' | sort -n -u | awk '{printf("%s,",$0);}'
```

还可以指定对某个文件的第几行第几列做上面的操作:

```
    sed -n '174p' config.ini | awk '{print $3}' | awk -F ',' '{for(i=1;i<=NF;i++) print $i}' | sort -n -u | awk '{printf("%s,",$0);}' > /tmp/nnq_uid.txt
```

计算号码个数：

```
    sed -n '174p' config.ini | awk '{print $3}' | awk -F ',' '{for(i=1;i<=NF;i++) print $i}' | sort -n -u | wc -l
```

## ubuntu上chkconfig的替代品sysv-rc-conf的基本使用

```
sudo apt-get install sysv-rc-conf 
sudo sysv-rc-conf
```

空格键切换服务的状态，q键退出 

## linux上使用rz和sz

假设当前机器叫A，远端机器叫B 

1. 首先A和B都要安装librzsz，

2. 机器A要安装zssh，使用方法跟ssh一样，如果是在gnome-connection-manager里使用，要修改他的代码：

``` 
sudo subl /usr/share/gnome-connection-manager/ssh.expect 第35行里的ssh改成zssh 
sudo subl /usr/share/gnome-connection-manager/gnome_connection_manager.py 第209行的ssh改成zssh
```

3. 然后用zssh连接到B。

    如果要从B传文件到A，要先在B执行sz filename，然后出现一串乱码后，接着按ctrl+2会切换到zmodem，这时敲入命令就相当于在A输入命令了，可以输入ls等命令，接着输入rz回车就可以开始传输了。

    如果要从A传文件到B，直接在B上按ctrl+2,然后sz filename就可以了。 

## 修改普通用户默认的fd limit

`vim /etc/security/limits.conf`

添加如下两行：

```
* soft nofile 1024000 
* hard nofile 1024000
``` 

## tmux技巧

配置文件路径 ~/.tmux.conf

设置快捷键，向嵌套的第二层发送前缀：   bind-key -n 'C-a' send-keys C-b C-b

设置vi mode： setw -g mode-keys vi    然后就可以[进入复制模式，space开始选择文本，回车进行复制，然后]粘贴                           

## 强制同步时间

```
service ntpd stop 
ntpdate us.pool.ntp.org 
service ntpd start
```

## 授权非root进程绑定小于1024端口

`setcap cap_net_bind_service=+ep /data/work/fsi/release/gnrl/conn_svr/conn_svr` 

http://stackoverflow.com/questions/413807/is-there-a-way-for-non-root-processes-to-bind-to-privileged-ports-1024-on-l 

这种方法设置只对当前文件生效（类似chmod），重新编译安装svr后又要重新授权 

## 设置tcp最大接收发送缓存

```
echo 51200000 > /proc/sys/net/core/wmem_max 
echo 51200000 > /proc/sys/net/core/rmem_max
```

## mysql里查询网络序IP的字符串：

INET_NTOA 

## 往磁盘里写一堆数据：

dd if=/dev/zero of=/data/swap/swap.20G bs=1024 count=20971520

## iptables做本机端口转发

`iptables -t nat -A PREROUTING -p tcp --dport 13357 -j REDIRECT --to-ports 3306`

更多例子看这里：http://blog.csdn.net/zzhongcy/article/details/42738285