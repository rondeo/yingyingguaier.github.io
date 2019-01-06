---
title: ubuntn踩坑手册
date: 2018-10-01 15:07:48
tags: [疑难杂症]
categories: ubuntu

---

# ubuntn踩坑手册

------

## 目录

1. [安装主题后Ctrl+Alt+T 终端快捷方式无效](#001 )
2. [安装burpsuite](#002)
3. [安装有道词典](#003)
4. [截图工具 Shutter](#004)
5. [安装DeepinScrot](#005)
6. [Ubuntu18.04双击网易云音乐无法启动](#006)
7. [Ubuntu18.04设置合上盖子时不进入休眠](#007)

<span id="001"></span>

## 安装主题后Ctrl+Alt+T 终端快捷方式无效

解决方案:手动添加快捷方式。

<span id="002"></span>
## Ubuntu中安装burpsuite

[参考文章(鸣谢)：一纸笔墨-Ubuntu中安装burp suite](https://www.jianshu.com/p/3e7bb41a1464)

### 安装java环境

java 8 下载地址：

https://www.oracle.com/technetwork/cn/java/javase/downloads/jdk8-downloads-2133151-zhs.html

选择`jdk-8u181-linux-x64.tar.gz`(当前最新的64版本,32位下载对应版本)。

下载完成后,以root用户进入下载目录,将下载到的`jdk-8u181-linux-x64.tar.gz`解压:`tar zxvf jdk-8u181-linux-x64.tar.gz`

将解压的得到的`jdk1.8.0_181`文件夹复制到/opt目录:`mv jdk1.8.0_181 /opt/`

> /opt目录用来安装附加软件包，是用户级的程序目录，可以理解为D:/Software。安装到/opt目录下的程序，它所有的数据、库文件等等都是放在同个目录下面。opt有可选的意思，这里可以用于放置第三方大型软件（或游戏），当你不需要时，直接rm -rf 掉即可。在硬盘容量不够时，也可将/opt单独挂载到其他磁盘上使用。

进入/bin目录(`cd /bin`)，并在/bin目录下创建java软链接：`ln -s /opt/jdk1.8.0_181/bin/java java`

> bin为binary的简写，/bin目录主要放置系统的必备执行文件，例如: cat、cp、chmod df、dmesg、gzip、kill、ls、mkdir、more、mount、rm、su、tar等。

在/bin目录创建了一个Java的软链接使我们都能使用Java。

> ln是linux中又一个非常重要命令，它的功能是为某一个文件在另外一个位置建立一个同步的链接.当我们需要在不同的目录，用到相同的文件时，我们不需要在每一个需要的目录下都放一个必须相同的文件，我们只要在某个固定的目录，放上该文件，然后在 其它的目录下用ln命令链接（link）它就可以，不必重复的占用磁盘空间。
>
> -s 软链接(符号链接)
>
> 软链接：
>
> 1.软链接，以路径的形式存在。类似于Windows操作系统中的快捷方式
>
> 2.软链接可以 跨文件系统 ，硬链接不可以
>
> 3.软链接可以对一个不存在的文件名进行链接
>
> 4.软链接可以对目录进行链接

进入/etc目录,修改/etc/profile文件，在文件末尾加上四行：

`export JAVA_HOME=/opt/jdk1.8.0_112`#注意版本

`export JRE_HOME=${JAVA_HOME}/jre`

`export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib`

`export PATH={JAVA_HOME}/bin:$PATH`

> /etc/profile文件用于修改环境变量，在这里修改的内容是对所有用户起作用的

最后使用命令`source /etc/profile`重新执行文件使环境变量生效，之后输入`echo $JAVA_HOME`，如果出现`/opt/jdk1.8.0_181`则说明安装应该是成功了。

### 安装burpsuite

从52pojie下载burpsuite,将解压得到的burp-loader-keygen.jar和burpsuite_pro_v1.7.37.jar文件移动到`/opt/burpsuite`(没有就创建一个)。在该目录下用`sudo java -jar burp-loader-keygen.jar`来运行注册机,用注册机登录。但是这样不方便,有以下解决方案。

> **/usr/bin**　是在后期安装的一些软件的运行脚本

```
cd /opt/burpsuite
sudo +x burp-loader-keygen.jar burpsuite_pro_v1.7.37.jar 
sudo vim /usr/bin/burpsuite
    #!/bin/bash
    java -Xbootclasspath/p:/opt/burpsuite/burp-loader-keygen.jar -jar /opt/burpsuite/burpsuite_pro_v1.7.37.jar

sudo chmod +x /usr/bin/burpsuite
```

之后就可以直接输入`sudo burpsuite`就可以运行了。

<span id="003"></span>
## 安装有道词典

https://github.com/yomun/youdaodict_5.5

### 有道词典 v1.1.1 ( 支持 PyQt 5.5 或以上 )

这个有道词典是从 Deepin 15.4.1 (youdao-dict_1.0.8-1_amd64.deb) 取出, 然后重新打包成 Ubuntu 能用的 deb 安装包
它支持 Ubuntu 16.10 / Fedora 24 / OpenSUSE 42.2 等开始的 Linux 分发版

<http://packages.deepin.com/deepin/pool/main/y/youdao-dict/>

1. 取得有道词典安装包 (需要3D加速, 假设显卡驱动已安装了)

```
wget https://github.com/yomun/youdaodict_5.5/raw/master/youdao-dict_1.1.1-0~ubuntu_amd64.deb
```

1. 如有 pip3 安装的 PyQt5, 卸载之.. (5.6 开始的版本, 因它缺所需旧模块)

```
如有显示 PyQt5 (5.10.1), 卸载之..
$ pip3 list | grep PyQt5
$ pip3 uninstall pyqt5

root 户口也一样要
$ sudo pip3 list | grep PyQt5
$ sudo pip3 uninstall pyqt5
```

1. 安装依赖软件包

- Ubuntu 16.04 - 18.04 / Debian 9.1 / Linux Mint 18.2 / Zorin OS 12.1

```
sudo su
$ apt install python3

$ apt install python3-dbus python3-lxml python3-pil python3-requests python3-xdg python3-xlib
$ apt-get install -f -y
$ apt install python3-pyqt5 python3-pyqt5.qtmultimedia python3-pyqt5.qtquick python3-pyqt5.qtwebkit

$ apt install gir1.2-appindicator3-0.1 qml-module-qtgraphicaleffects qml-module-qtquick-controls
$ apt install libqt5multimedia5-plugins ttf-wqy-microhei
$ apt install tesseract-ocr tesseract-ocr-eng tesseract-ocr-chi-sim tesseract-ocr-chi-tra

$ apt install ubuntu-restricted-extras

我这里python3-lxml没安装上,单独来一次。
$ apt-get install python3-lxml
$ dpkg -i youdao-dict_1.1.1-0~ubuntu_amd64.deb
上面的没问题到这步就成功了。
Ubuntu 18.04 用 fonts-wqy-microhei 取代了 ttf-wqy-microhei
```
<span id="004"></span>
## 截图工具Shutter

ubuntu 安装截图工具 Shutter，并设置快捷键 Ctrl+Alt+A

### 安装截图工具 Shutter

```
$ sudo apt-get install shutter
```

### 设置 Shutter 截屏快捷键 [ctrl + Alt + A]

1. Ubuntu右上角右键 - 打开系统设置 - 打开键盘 - 点击快捷键 - 自定义快捷键
2. 点击 + 号， 添加快捷键。 名称随便命名，如 shutter select, 命令 设置为 shutter -s 
   点击应用 - 点击 Disabled - 然后迅速按下 ctrl + alt + a

<span id="005"></span>

## 安装DeepinScrot

DeepinScrot : 这是一个在深度(Deepin)操作系统中使用的截图工具。 
深度操作系统是中国国产的基于Linux的操作系统, 界面还蛮清新的, 由武汉深之度科技有限公司研发。

DeepinScrot截图工具类似于QQ,适合国人。

### 安装

```
wget http://packages.linuxdeepin.com/deepin/pool/main/d/deepin-scrot/deepin-scrot_2.0-0deepin_all.deb
sudo dpkg -i deepin-scrot_2.0-0deepin_all.deb 
sudo apt-get install -f
#重新
sudo dpkg -i deepin-scrot_2.0-0deepin_all.deb 
#安装成功
```

### 使用DeepinScrot

```
deepin-scrot 
```

### 设置DeepinScrot快捷键

　 ubuntu 设置-键盘－自定义快捷键 
　 填写名称，命令(deepin-scrot)，快捷键 

<span id="006"></span>

## Ubuntu18.04双击网易云音乐无法启动

解决方案来自知乎。

Ubuntu 18.04 装了网易云音乐，难道只能用 sudo 启动吗？ - Fancy的回答 - 知乎
https://www.zhihu.com/question/277330447/answer/478510195

### 问题描述

只能在命令行下用`sudo netease-cloud-music`启动网易云音乐,双击网易云音乐无法启动。

### 问题产生原因

双击图标无法启动是因为环境变量 SESSION_MANAGER在捣乱。sudo后该环境变量变为了空。

```
#输出为空
sudo env|grep SESSION_MANAGER
#有输出 
env|grep SESSION_MANAGER
```

### 解决方案

```
# 对应行修改为 Exec=sh -c "unset SESSION_MANAGER && netease-cloud-music %U" 
sudo vim /usr/share/applications/netease-cloud-music.desktop
# 修改Exec=
sh -c "unset SESSION_MANAGER && netease-cloud-music"
```

![006_1](/img/UbuntuPro/006_1.png)

保存并退出后即可双击打开网易云音乐。

<span id="007"></span>

## Ubuntu18.04设置合上盖子时不进入休眠

### 问题描述：

有的时候想要笔记本合上盖子也能继续工作,但是没有设置过的笔记本合上盖后工作环境就停下来了。这就需要进行相关配置。

### 解决方案

`/etc/systemd/logind.conf`:登录管理配置文件

```
sudo vim /etc/systemd/logind.conf
#修改处
HandleLidSwitch=ignore
```

保存并退出,还需要重启服务生效。

`sudo restart systemd-logind `

下面是登录管理配置文件的一些细节

HandlePowerKey=, HandleSuspendKey=, HandleHibernateKey=, HandleLidSwitch=, HandleLidSwitchDocked=

当 power(电源)/sleep(休眠)/lid(合上盖子) 事件发生时， 应该执行何种操作： 

"ignore"(无操作), "poweroff"(关闭系统并切断电源), "reboot"(重新启动), "halt"(关闭系统但不切断电源), "kexec"(调用内核"kexec"函数), "suspend"(休眠到内存), "hibernate"(休眠到硬盘), "hybrid-sleep"(同时休眠到内存与硬盘), "lock"(锁屏) 。 

注意， 只监视带有 "power-switch" 标签的 输入设备的 key(按下按钮)/lid(合上盖子) 事件。 如果主机插入了一个扩展坞(docking station) 或者连接了多个显示器， 那么"合上盖子"将执行 HandleLidSwitchDocked= 动作， 否则将执行 HandleLidSwitch= 动作。 

下面是各选项的默认值： 

HandlePowerKey=poweroff 、 HandleSuspendKey=suspend 、 HandleLidSwitch=suspend 、 

HandleLidSwitchDocked=ignore 、 HandleHibernateKey=hibernate

