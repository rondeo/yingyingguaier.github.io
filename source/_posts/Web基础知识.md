---
title: Web基础知识
date: 2019-01-05 11:18:24
tags: 
---

# 基础知识

------

## 常见的端口

|      端口号       |     服务      |             漏洞              |
| :---------------: | :-----------: | :---------------------------: |
|        21         |      ftp      |                               |
|        22         |      ssh      |                               |
|        23         |    telnet     |                               |
|        161        |     snmp      |                               |
|        443        |    openssl    |           心脏滴血            |
|        837        |     Rsync     |            未授权             |
|       1433        |    MSSSQL     |                               |
|       1521        |    Oracle     |                               |
|       2375        |    docker     |            未授权             |
|       3306        |     Mysql     |                               |
|       3389        |   远程连接    |                               |
|       5432        |  PostgreSQL   |                               |
|       5900        |      vnc      |                               |
|       6379        |     redis     |            未授权             |
|  7001,7002,7003   |   Weblogic    |      默认弱口令，反序列       |
|       9000        |  fastcgi-php  |                               |
|     9200,9300     | elasticsearch |  CVE-2015-1427,CVE-2014-3120  |
|       11211       |   memcache    |            未授权             |
|    27017,27018    |    Mongodb    |          未授权访问           |
|       1099        |      rmi      |           命令执行            |
|     2601,2604     |     zebra     |     路由，默认密码 zebra      |
|       3128        |     squid     |         代理默认端口          |
|       4440        |    rundeck    |                               |
|       4848        |   glassfish   | 中间件弱口令 admin/adminadmin |
|       5984        |    CouchDB    |    http://xxx:5984/_utils/    |
|       8000        |   jdwp java   |       调试接口命令执行        |
|       9043        |   websphere   |      弱口令 admin/admin       |
|       50000       |      SAP      |           命令执行            |
| 50060,50070,50030 |    hadoop     |      默认端口未授权访问       |

## SQL注入类型

数字型，字符型，搜索型

## 联合查询的条件（union select）

1. 页面有注入回显
2. 拼接的两个结果集必须有相同列数,列的数据类型相同

## DNSlog知道吗？是否能在Linux利用

https://www.anquanke.com/post/id/98096

![001](/img/knowledge/web/001.jpg)

load_file函数在Linux下是无法用来做dnslog攻击的，因为在这里就涉及到Windows的UNC路径

## 绕过waf的方法

https://www.freebuf.com/column/163469.html

## 利用过参数污染绕过waf吗？

在跟服务器交互的过程中，http允许 get 或者post多次传同一参数值，造成覆盖达到一些绕过waf的效果。

```
GET /foo?par1=val1&par2=val2 HTTP/1.1
User-Agent: Mozilla/5.0
Host: Host
Accept: */*
```

如上面的http请求，一般同一参数名字只能传参一次，但是http协议中允许同样名称的参数出现多次。

不同的服务器处理结果会不一样，利用这个特性进行绕过。

## XSS类型

反射型，存储型，DOM型

## xss可利用的协议

javascript 伪协议，data 协议，MHTML注射XSS

## 同源策略

3同，同域名同端口同协议。作用就是防止文档或脚本从不同源加载。

## CSRF

定义：攻击者劫持用户访问web站点执行非用户本意的操作。

完成csrf的即可条件：

1. 受害者处于登录状态
2. 诱使用受害者点击连接
3. 系统对敏感操作没有进行验证

## 常见的请求方式有哪些

post,get,options,head,connect,trace,delete,put

## ssrf利用协议

最具代表性几个，dict操作redis,file任意文件读，gopher反弹shell，tftp发起udp拒绝服务

## ssrf无回显怎么办

利用dnslog

## ssrf如何绕过方式

1）[http://www.baidu.com@10.10.10.10与http://10.10.10.10](http://www.baidu.com@10.10.10.xn--10http-4p7i//10.10.10.10) 请求是相同的

此脚本访问请求得到的内容都是10.10.10.10的内容。

该绕过同样在URL跳转绕过中适用。

<http://www.wooyun.org/bugs/wooyun-2015-091690>

> 乌云镜像的url中的html替换为wooyun-2015-091690.html

2）ip地址转换成进制来访问

115.239.210.26 ＝ 16373751032

3）添加端口可能绕过匹配正则

10.10.10.10:80 案例：

<http://www.wooyun.org/bugs/wooyun-2014-061850>

> 乌云镜像的url中的html替换为wooyun-2014-061850.html

4）localhost绕过

5）用短地址（302跳转）绕过，案例:

搜狐某云平台API接口SSRF续集-绕过:https://bugs.shuimugan.com/bug/view?bug_no=0132243

神器而已之奇虎360某SSRF绕过限制代理访问内网:https://bugs.shuimugan.com/bug/view?bug_no=0135257

5）利用xip.io和xip.name

```
10.0.0.1.xip.io 10.0.0.1

www.10.0.0.1.xip.io 10.0.0.1

mysite.10.0.0.1.xip.io 10.0.0.1

foo.bar.10.0.0.1.xip.io 10.0.0.1
10.0.0.1.xip.name  resolves to  10.0.0.1

www.10.0.0.2.xip.name  resolves to  10.0.0.2

foo.10.0.0.3.xip.name  resolves to  10.0.0.3

bar.baz.10.0.0.4.xip.name  resolves to  10.0.0.4
```

6）利用Enclosed alphanumerics

```
利用Enclosed alphanumerics
ⓔⓧⓐⓜⓟⓛⓔ.ⓒⓞⓜ  >>>  example.com
List:
① ② ③ ④ ⑤ ⑥ ⑦ ⑧ ⑨ ⑩ ⑪ ⑫ ⑬ ⑭ ⑮ ⑯ ⑰ ⑱ ⑲ ⑳ ⑴ ⑵ ⑶ ⑷ ⑸ ⑹ ⑺ ⑻ ⑼ ⑽ ⑾ ⑿ ⒀ ⒁ ⒂ ⒃ ⒄ ⒅ ⒆ ⒇ ⒈ ⒉ ⒊ ⒋ ⒌ ⒍ ⒎ ⒏ ⒐ ⒑ ⒒ ⒓ ⒔ ⒕ ⒖ ⒗ ⒘ ⒙ ⒚ ⒛ ⒜ ⒝ ⒞ ⒟ ⒠ ⒡ ⒢ ⒣ ⒤ ⒥ ⒦ ⒧ ⒨ ⒩ ⒪ ⒫ ⒬ ⒭ ⒮ ⒯ ⒰ ⒱ ⒲ ⒳ ⒴ ⒵ Ⓐ Ⓑ Ⓒ Ⓓ Ⓔ Ⓕ Ⓖ Ⓗ Ⓘ Ⓙ Ⓚ Ⓛ Ⓜ Ⓝ Ⓞ Ⓟ Ⓠ Ⓡ Ⓢ Ⓣ Ⓤ Ⓥ Ⓦ Ⓧ Ⓨ Ⓩ ⓐ ⓑ ⓒ ⓓ ⓔ ⓕ ⓖ ⓗ ⓘ ⓙ ⓚ ⓛ ⓜ ⓝ ⓞ ⓟ ⓠ ⓡ ⓢ ⓣ ⓤ ⓥ ⓦ ⓧ ⓨ ⓩ ⓪ ⓫ ⓬ ⓭ ⓮ ⓯ ⓰ ⓱ ⓲ ⓳ ⓴ ⓵ ⓶ ⓷ ⓸ ⓹ ⓺ ⓻ ⓼ ⓽ ⓾ ⓿
```

7) 利用句号

```
127。0。0。1  >>>  127.0.0.1
```

## 文件包含有哪些伪协议

https://www.yingyingguaier.top/2019/01/03/php%20%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E4%BC%AA%E5%8D%8F%E8%AE%AE%E5%88%A9%E7%94%A8/#more

## 文件包含如何getshell

1. 远程文件包含：可以直接执行任意代码

   ------

   下面是本地包含

   ------

2. 上传图片马包含

3. 包含日志文件Getshell

4. 利用data伪协议和php://input流执行php代码

5. 存在phpinfo信息泄露可包含临时文件

6. 包涵session文件


