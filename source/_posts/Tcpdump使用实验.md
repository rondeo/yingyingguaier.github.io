---
title: Tcpdump使用实验
date: 2018-09-20 09:09:12
tags:
categories: 工具
---
# <center>Tcpdump使用实验</center> # 

## 【目的】 ##

学习使用tcpdump工具，tcpdump是最常用的网络数据分析工具之一。

## 【原理】 ##

tcpdump和wireshark一样,是一个抓包工具,基于libpcap开发的,过滤机制使用内核的BPF过滤,在linux下使用,也可以把tcpdump当成linux的命令。多数linux服务器是不按照图形界面的，在这种情况下，就可以使用tcpdump。

## 【步骤】 ##

### 安装 ###


tcpdump一般在linux服务器上是默认安装的,也可以说tcpdump是linux服务器端一个命令,可以用whereis命令查看安装位置;

![1](https://i.imgur.com/i32BldG.png)

用`tcpdump -help`查看帮助	

![2](https://i.imgur.com/cbMrq2X.png)

### 常用命令选项 ###

> 跟进不同用户的权限,如果需要sudo的,在命令前面加上sudo
	
#### 指定要抓的数据包的数量 ####

指定要抓的数据包的数量,指定"-c 10"将获取10个包,但可能已经处理了100个包,只不过只有10个包是满足条件的包。

`tcpdump -c 10`

![3](https://i.imgur.com/maAUOo6.png)

#### 查看哪些端口可以抓包 ####
	
`tcpdump -D`

![4](https://i.imgur.com/OgWm7o4.png)

#### 指定接口 ####

`tcpdump -i eth0 -c 10`

![5](https://i.imgur.com/JYZyM0u.png)

#### -n和-nn 

-n: 对地址以数字方式显式,否则显式为主机名,也就是说-n选项不做主机名解析

-nn: 把端口显示为数值,否则显示端口服务名

对比下上图`tcpdump -i eth0 -c 10`红色方框的部分

`tcpdump -i eth0 -c 10 -n`

`tcpdump -i eth0 -c 10 -nn`

![6](https://i.imgur.com/IeFHwOL.png)

#### -x和-xx;-X和-XX,最常用的是-XX

-x：以16进制打印出每个包的数据(不包括连接层的头部)

-xx：以16进制打印出每个包的数据

-X：输出包头部的数据,以16进制和ASCII两种方式同时输出（不包括连接层的头部）

-XX：输出包头部的数据,会以16进制和ASCII两种方式同时输出。
	
![7](https://i.imgur.com/ozN3Uef.png)

![8](https://i.imgur.com/PjSKRna.png)

#### -v，-vv，-vvv答应详细输出

一个比一个详细，看一下吧

如果没带-v或者-vv选项的时候，会有如下提示

![9](https://i.imgur.com/9hsei8t.png)

对比一下带-v和不带-v的（`tcpdump -i eth0 -c 10 -nn`）区别，会多输出一些信息。

![10](https://i.imgur.com/Ko5X6wn.png)

#### 包保存到指定文件-w，从指定文件读取包答应到屏幕-r

![11](https://i.imgur.com/JNiPpWS.png)

![12](https://i.imgur.com/lY5hUnm.png)

### 常用过滤字段 ###

#### 过滤指定主机 ####

首先ping下，看看ip是多少，然后用host过滤，抓到的都是指定host的包

`tcpdump host 360.cn`

![13](https://i.imgur.com/Hz3k3qH.png)

#### 源与目的，src与dst ####

`tcpdump -i eth0 -c 10 -nn src host 360.cn`

`tcpdump -i eth0 -c 10 -nn dst host 360.cn`

![14](https://i.imgur.com/4iM4VTS.png)

`tcpdump -i eth0 -c 10 -nn src host 36.110.213.49`

`tcpdump -i eth0 -c 10 -nn dst host 36.110.213.49`

![15](https://i.imgur.com/MLaCZEj.png)

#### 协议过滤 ####

过滤udp协议

`tcpdump -i eth0 -c 10 -nn udp`

![16](https://i.imgur.com/ea8zQXs.png)

抓ping包

`tcpdump -i eth0 -c 10 -nn icmp`

![17](https://i.imgur.com/W8602BO.png)

#### 过滤网段 ####

`tcpdump -i eth0 -c 10 -nn net 10.60`

![18](https://i.imgur.com/2Y9swSM.png)

#### 过滤端口 ####

`tcpdump -c 5 udp port 53`

![19](https://i.imgur.com/P6iJhc3.png)

#### 协议字段过滤 ####

表达式单元之间可以使用操作符" and / && / or / || / not / ! "进行连接

过滤syn包和fin包

`tcpdump -i eth0 -c 3 -nn 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0'`

![20](https://i.imgur.com/vyNC7VE.png)

过滤tcp 80端口，ip包长度大于1000的包（ip[2:2]表示整个ip数据包的长度）

`tcpdump -i eth0 -c 1 -nn 'tcp port 80 and ip[2:2] > 1000'`

![21](https://i.imgur.com/OFtQNEF.png)

过滤icmp的reply包

`tcpdump -i eth0 -c 1 -nn 'icmp[icmptype] == icmp-echoreply'`

![22](https://i.imgur.com/7PLTIWu.png)

