---
title: sqli-labs-lession 16 POST双引号布尔型时间盲注
date: 2018-09-27 22:15:07
tags: [sqli-labs]
categories: sql注入
---
# sqli-labs-lession 16 POST双引号布尔型时间盲注 #

## 登录界面 ##

![1](/img/sql/Lesson-16/1.png)

## 注入 ##

和lession-15一样是布尔型时间盲注,只是闭合不同

### 判断类型 ###

`admin") and if(1,sleep(60),null)#`

测试出类型为字符型

![2](/img/sql/Lesson-16/2.png)


### 参考 ###

{% post_link sqli-labs-lession-9-GET单引号基于时间盲注 查看Lesson-9%}

