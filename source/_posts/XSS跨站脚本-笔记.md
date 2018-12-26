---
title: XSS跨站脚本-笔记
date: 2018-12-25 21:13:14
tags: [XSS]
categories:
  - [XSS]
  - [整理]
---

# XSS跨站脚本-笔记

---

## XSS危害

```shell
1.网络钓鱼,盗取用户账号
2.窃取用户cookie,利用用户身份进一步对网站执行操作
3.强制弹广告,刷流量
4.网页挂马  
5.客户端攻击,DDOS等
6.获取客户端信息,如浏览历史,真实ip,端口
7.结合csrf
8.提升用户权限,进一步渗透网站
9.跨站脚本蠕虫
等
```

### 如何利用XSS提权?

如果后台存在XSS,当管理员触发时,可以利用JavaScript发送请求,该请求携带管理员的Cookie发送到服务器,在后台添加一个新管理员用户。（涉及Ajax技术）

### 利用XSS获取Webshell?

1.特殊情况：Access+ASP架构，网站管理员出于某些原因把数据库文件后缀修改成ASP。

网站存在存储型XSS，攻击者可写入一句话木马来getshell.

2.后台存在存储型XSS，且后台具有数据库备份功能。攻击者上传含有一句话木马的图片，通过XSS劫持管理员会话将上传的图片重备份为Webshell。

### 网络钓鱼

钓鱼页面由以下几个部分组成

1. 钓鱼页面，经常是登录表单部分
2. 记录受害者登录信息的脚本
3. XSS phishing Expliot :在有xss的页面插入利用代码
iframe框架的巧妙利用，覆盖原页面，迷惑性强

### 如果设置了Httponly如何利用XSS

1. 直接获取用户的明文账户密码信息，监听键盘记录

2. 进行会话劫持

   https://www.freebuf.com/articles/web/129384.html

### XSS获取客户端信息

1. 利用JS获取CSS信息（CSS信息能定义访问过和未访问过的超链接样式）

2. JS实现漏洞扫描
3. 获取剪贴板内容
4. 获取IP地址

### XSS如何网页挂马？

攻击者通过篡改网页来实现，最常见的案例是利用一个长宽属性为0的`<iframe>`标签来加载木马页面。或者用JS动态调用网页木马。

### XSS发动DOS/DDOS的情况？

利用XSS恶意代码对网站不断发送请求，还可以劫持大量浏览器来实施攻击。

### DOM型XSS的特殊性

DOM型虽然和反射型类似，都需要诱使用户点击url，但是DOM型的url不需要经过服务器，在客户端浏览器就能执行。

### 同源策略

同协议，同域名，同端口，可以防止从不同域装载文档或脚本。

跨域方法：JSONP,Iframe,Flash

### XSS结合

#### CSRF

CSRF攻击能够劫持终端用户在已登录的Web站点上执行非本意的操作。

案例：iGaming CMS后台存在CSRF漏洞，后台利用Requests接收，可以改造添加管理元的url诱使管理员访问。

### CRLF注射

利用单个\r\n可以注入会话cookie(会话固定)，两个\r\n可以注入HTML/JavaScript代码。

案例：某校园网登录界面存在CRLF注射，利用两个%0d%a可以执行js代码。

#### MHTML协议

格式：mhtml:http://127.0.0.1/demo.html!demo

http://127.0.0.1/demo.html 对应 【Mhtml_File_Url】

demo 对应 【Original_Resource_Url】

【Original_Resource_Url】从消息体的Content-Location获得，如果不能从【Mhtml_File_Url】获得，会主动去【Original_Resource_Url】取获取下载内容。

案例：豆瓣搜索文本出现过MHTML协议代码注射问题，能够执行XSS。

#### DATA URI

常用来绕过XSS的过滤

```html
<a href=data:text/html;base64,PHNjcmlwdD5hbGVydCgzKTwvc2NyaXB0Pg==>
```

## 防御XSS

### 输入过滤

#### 输入验证

控制输入的格式

#### 数据消毒
过滤敏感字符

```
< > ' " & # javascript expression
```

### 输出编码

对敏感字符镜像转义，将HTML特殊字符转成字符实体编码。

### 黑名单和白名单

### DOM型

1. 避免客户端文档重写、重定向或其他敏感操作
2. 分析和强化客户端JS代码，尤其是一些受用户影响的DOM对象。Referrer属性，windowsname属性等