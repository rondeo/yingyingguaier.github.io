---
title: Windows下PHP环境安装
date: 2018-09-19 09:09:12
tags:
categories: php
---

#  windows下PHP环境的搭建 #

---

## 概述 ##
 	
windows下的PHP环境简称WAMP（windows+Apache+MySQL+PHP）

PHP的环境搭建需要安装Apache+MySQL+PHP,还需要进行相关配置。与集成开发环境相比,WAMP的过程相对繁琐,那么可以使用phpstudy这样的开发环境。

下面MySQL数据库使用用MariaDB,它是MySQL源代码的一个分支,与MySQL使用基本相同,在MySQL被甲骨文收购后随时存在闭源的风险的情况下,他是一个很好的替代品。

下载地址:

* [Apache](https://httpd.apache.org/download.cgi#apache24)
* [PHP](http://www.php.net/) 
* [MariaDB](https://downloads.mariadb.org/)

安装顺序按照Apache--->PHP--->MySQL进行。

## 安装包准备 ##

### Apache ###

点击上面的下载地址进入Apache官网

![1](https://i.imgur.com/7zAnKFy.png)

![2](https://i.imgur.com/edi2Whn.png)

![3](https://i.imgur.com/uKZSvyc.png)

根据你的windows操作系统位数选择安装包

### PHP ###

点击上面的下载地址进入PHP官网

![4](https://i.imgur.com/EUeJCvu.png)

选择最新版链接下载进入PHP官网

![5](https://i.imgur.com/Jo2tRgC.png)

![6](https://i.imgur.com/7m2gZiA.png)

根据你的windows操作系统位数选择安装包

### MariaDB ###

点击上面的下载地址进入MariaDB官网

![7](https://i.imgur.com/q3BIl6H.png)

![8](https://i.imgur.com/FTeAPYX.png)

根据你的windows操作系统位数选择安装包

### 结果 ###

![9](https://i.imgur.com/1OWP6zr.png)

## Apache 安装 ##

> 将http开头的安装包解压,多出一个`Apache24`的文件夹

![10](https://i.imgur.com/Vcy2Nmm.png)

> 将`Apache24`放在你想要放的任何位置,为了方便管理,我在C盘新建一个目录`WAMP`,把`Apache24`放进去。之后也把所有的安装放在这里

![11](https://i.imgur.com/RvNhCf4.png)

进入到`Apache24\bin`

![12](https://i.imgur.com/cU7NZIa.png)

<font color=red>
	将上图地址栏标记的的地址复制
	
	`C:\WAMP\Apache24\bin`
</font>

![13](https://i.imgur.com/PzSG47T.png)

鼠标移到`cmd`上,右键以管理员身份运行

![14](https://i.imgur.com/q3TSCMY.png)

输入`cd `(注意这有个空格) 并把刚才复制的`C:\WAMP\Apache24\bin`粘贴上去,回车运行

![15](https://i.imgur.com/UyPn696.png)

可以看到路径改变了,再输入`httpd.exe -k install`,执行安装过程。

![16](https://i.imgur.com/ULVsMlk.png)

如果出现了上图的错误,是因为Win7系统是缺少了VC++2015组件,Win10系统就不会出现这个问题。

cmd窗口先别关闭。点下面的连接下载组件

[VC++2015组件](https://www.microsoft.com/zh-CN/download/details.aspx?id=48145)

![17](https://i.imgur.com/2A5ZNpn.png)

我是Win7 32位系统就选择32位的(64位就选64位的),然后点next。

把下载好的安装程序双击运行。同意安装直到安装成功。

![18](https://i.imgur.com/0YLPYL3.png)

重新输入`httpd.exe -k install`,你也可以按方向键的↑,它记录了历史输入,找到`httpd.exe -k install`后回车执行。

![19](https://i.imgur.com/Ndq3QvL.png)

提示Apache2.4安装成功。

安装成功后按Windows+R键,输入`services.msc`快捷打开系统服务。

![20](https://i.imgur.com/JUpD4GF.png)

找到`Apache2.4`

![21](https://i.imgur.com/k1y7ybF.png)

点击后右键属性修改启动类型为手动。

![22](https://i.imgur.com/P5cwtUi.png)

点击启动后,Apache2.4会运行。

如果你出现和我下图一样的错误,可以查看解决办法,没有出现该问题可不看这步

![23](https://i.imgur.com/QADrjLI.png)

在我的安装目录`C:\WAMP\Apache24\bin`下有个`httpd.exe`,用cmd让他执行,它会提示错误信息。

![24](https://i.imgur.com/z8AJr2m.png)

![25](https://i.imgur.com/eigfN7c.png)

<font color=red>
错误原因:httpd.conf里面配置的ServerRoot路径跟实际路径不一致，导致路径无效。
</font>

在你的安装路径的`Apache24\`底下有个`conf`文件夹,进入到该目录。有个`httpd.conf`文件。`httpd.conf`文件包含了Apache的配置。用编辑器打开`httpd.conf`,我用的是EditPlus,什么编辑器都没有的话就用自带的记事本打开。

![26](https://i.imgur.com/QfySGgM.png)

找到提示错误的39行

![27](https://i.imgur.com/S5nUOVU.png)

其实是38行需要修改,修改成如下图配置,它其实记录了`Apache24\`的路径,它默认配置是在C盘根目录的`Apache24\`,一旦你把`Apache24\`放在别的地方都需要改动这个配置。这里我添加上`C:\WAMP`,显示如下。也就是说如果你的路径在`D:\Apache24`下面就填`D:\Apache24`。

![28](https://i.imgur.com/nw4tNUj.png)

修改记得保存,启动Apache2.4成功

![29](https://i.imgur.com/EXJwtuV.png)

随便打开个浏览器输入

`http://localhost/`

如果返回如下界面就表示配置成功,Apache这一关就算过了。

![30](https://i.imgur.com/MxKHSo4.png)

## PHP 安装及配置 ##

### 解压 ###

将php的压缩解压,当前目录下多出了php开头的文件夹,重命名为php后拷贝到`C:\WAMP`

![31](https://i.imgur.com/X1jc6ey.png)

![32](https://i.imgur.com/rNmAYLv.png)

### 选择配置文件 ###
双击进入php文件夹,有两个配置文件`php.ini-development`和`php.ini-production`。

![33](https://i.imgur.com/1RTQN4W.png)

要使用哪个就把文件名修改成`php.ini`就行。`php.ini-production`在参数配置上更具有安全性,适合产品上线使用,学习环境下使用`php.ini-development`就行了。这里将`php.ini-development`改成`php.ini`。

用文本编辑器打开改名的`php.ini`。

### 开启扩展模块 ###

搜索`extension_dir`,大概在730行左右。

![34](https://i.imgur.com/9bi5KDp.png)

配置文件中`;`的作用是注释,要开启某个配置需要把开头的`;`去掉。
这里配置的是php的扩展目录,该目录文件为PHP提供一些扩展的功能。可以查看下该目录。

![36](https://i.imgur.com/AI7j3y3.png)

将该路径复制到“”中,并把开头启注释作用的`;`去掉。这里的路径填写你自己的路径。

![35](https://i.imgur.com/46N7GV9.png)

修改之后PHP就能找到扩展文件所在的目录了,但是这些找到的扩展文件没有启用,需要手动配置。现在启用mysql数据库扩展,在`php.ini`中搜索`mysqli`

![37](https://i.imgur.com/H4zCLcW.png)

大概在897行

`;extension=mysqli`

去掉`;`开启mysql数据库扩展

![38](https://i.imgur.com/rv8U8iZ.png)

保存并退出。

## Apache 连接 PHP ##

### 概述 ###

将Apache安装好和PHP安装配置好后并不表示Apache和PHP关联起来了,还要配置Apache的配置文件`httpd.conf`让Apache认识PHP。

### Apache加载PHP模块 ###

我的Apache配置文件在`C:\WAMP\Apache24\conf`下,用编辑器打开。

![39](https://i.imgur.com/5OHHMYx.png)

找到183行左右,这是加载模块的最后一行

![40](https://i.imgur.com/Ruuy221.png)

添加下面两句话
<p>LoadModule php7\_module "<font color=red>C:\WAMP\php</font>\php7apache2_4.dll"<p>

<p>PHPIniDir "<font color=red>C:\WAMP</font>\php"<p>

红色部分修改为你的路径

![41](https://i.imgur.com/oACAJzi.png)

第一句话加载了PHP模块

第二句添加了`php.ini`所在的目录

### Apache解析PHP###

找435行左右

![42](https://i.imgur.com/Gof1i6b.png)

添加

`AddHandler application/x-httpd-php .php`

![43](https://i.imgur.com/myPKwea.png)

保存并退出

重新启动Apache

![44](https://i.imgur.com/iD6KxBm.png)

### 测试是否正常解析PHP文件 ###

进入`Apache24\`下的`htdocs`目录,新建一个`phpinfo.php`的php文件

![45](https://i.imgur.com/70SPdg3.png)

编辑`phpinfo.php`内容

![46](https://i.imgur.com/XIGNqcV.png)

`<?php
	phpinfo();
?>`

删除该目录下的`index.html`文件,因为它会妨碍目录显示。访问`localhost`

![47](https://i.imgur.com/7rNLnrk.png)

打开`phpinfo.php`文件,显示如下则解析成功。

![48](https://i.imgur.com/IWz1Lw2.png)

## MYSQL 安装及测试 ##

### MYSQL安装 ###

![49](https://i.imgur.com/dkEp5De.png)

双击安装

![50](https://i.imgur.com/VVm8POP.png)

建立数据库用户,这里我建立账户名用户名都为root

![51](https://i.imgur.com/mphJYjl.png)

![52](https://i.imgur.com/YBbIiAH.png)

默认就行

### 测试数据库是否安装成功 ###

新建`mysql.php`

![53](https://i.imgur.com/9trRZo4.png)

下面的username填你刚才创建的用户名,password填你刚创建的密码。

数据库测试代码:

	<?php
		$servername = "localhost";
		$username = "root";
		$password = "root";
	 
		// 创建连接
		$conn = new mysqli($servername, $username, $password);
	 
		// 检测连接
		if ($conn->connect_error) {
	    	die("fail: " . $conn->connect_error);
		} 
		echo "success";
	?>

![54](https://i.imgur.com/9y23zfv.png)

双击运行

![55](https://i.imgur.com/dQtf4AN.png)

返回success表示数据库安装成功

![56](https://i.imgur.com/H0doqgq.png)

