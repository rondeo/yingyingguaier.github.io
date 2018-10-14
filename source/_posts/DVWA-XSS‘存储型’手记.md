---
title: DVWA-XSS‘存储型’手记
date: 2018-10-10 16:43:26
tags: [DVWA,XSS]
categories: XSS
---

# DVWA-XSS‘存储型’手记

---

## LOW

### Source:

![001](/img/xss/dvwa/Stored/low_001.png)

### Analysis:	

先对用户名和消息去除了两边的空格。将消息在输入时转义下存储进数据库。但是二次读取后没有检查,这里Name和Message都可以注入。直接用`<script>alert(1)</script>`弹窗测试。

在前端有对name有长度限制,可以F12开发者模式定位到进行修改。修改maxlength="100"后可以对name进行xss注入了。

![002](/img/xss/dvwa/Stored/low_002.png)

![003](/img/xss/dvwa/Stored/low_003.png)

```
<a href=# onmouseover=alert('Name')>ClickHere</a>
<a href=# onmouseover=alert('Message')>ClickHere</a>
```

### Process:

![004](/img/xss/dvwa/Stored/low_004.png)

## MEDIUM

### Source:

![005](/img/xss/dvwa/Stored/medium_001.png)

### Analysis:	

```
利用trim函数清除$message和$name 两端空格。
$message用strip_tags函数从字符串中去除 HTML 和 PHP 标记。后再用htmlspecialchars进行转义。把$message的路堵死了。
将$name中的<script>替换为空。

对script替换成空。可以采用多种方法绕过。
1. 双写
	<sc<script>ript>alert('双写')</script>
2. 添加空格或换行
	添加空格
	<script >alert('添加空格')</script>
	
	添加换行
	<script 
	>alert('添加换行')</script>
 3. 替换大小写
 	<Script>alert('修改S')</script>
 	
 先在前端修改对name的输入框属性maxlength="100"，再进行xss注入。
 也可以像下面的payload一样不使用<script>
 payload：
 <a href=# onmouseover=alert('Name')>ClickHere</a>
```

### Process:

1. 双写

![006](/img/xss/dvwa/Stored/medium_002.png)

2. 添加空格

![007](/img/xss/dvwa/Stored/medium_003.png)

​       		添加换行

![008](/img/xss/dvwa/Stored/medium_004.png)

3. 替换大小写

   ![009](/img/xss/dvwa/Stored/medium_005.png)

4. 不使用`<script>`

![010](/img/xss/dvwa/Stored/medium_006.png)

## HIGH

### Source:

![011](/img/xss/dvwa/Stored/high_001.png)

### Analysis:	

```
过滤了<script,检查了大小写。
对于这种情况不使用script就行了。
1.事件触发执行javascript
	<a href=# onmouseover=alert('事件')>ClickHere</a>
2.Src属性
	<img src=x onerror=alert('Src属性');>
照例先进行前端绕过Name的长度限制。
```

### Process:

1.事件触发执行javascript

![012](/img/xss/dvwa/Stored/high_002.png)

2.Src属性

![013](/img/xss/dvwa/Stored/high_003.png)

## IMPOSSIBLE

### Source:

![014](/img/xss/dvwa/Stored/impossible_001.png)

### Analysis:	

```
htmlspecialchars 将特殊字符转换为 HTML 实体
& (& 符号)	&amp;
" (双引号)	&quot;，除非设置了 ENT_NOQUOTES
' (单引号)	&#039; 
< (小于)	&lt;
> (大于)	&gt;

Impossible等级的代码使用htmlspecialchars函数把预定义的字符转换为HTML实体，
防止浏览器将其作为HTML元素。从而防治了XSS利用和危害。
```





