---
title: reGeorg+Proxifier渗透内网
date: 2019-06-19 22:58:24
tags: [内网渗透]
---

# reGeorg+Proxifier渗透内网

## 0x00 前言

渗透测试中拿到了一台Web服务器之后发现这台主机可以访问内网，可能还会进行内网渗透。这时候就需要通过这台主机来进行内网渗透，但是通过这台机器去渗透内网又需要上传很多工具，会很麻烦。这时可以把在这台服务器作为一个跳板，利用代理使我们访问内网。

## 0x02 工具

1. reGeory

   [reGeory](https://github.com/sensepost/reGeorg)是GitHub上的一个开源项目：https://github.com/sensepost/reGeorg

   主要由两部分组成：tunnel的各类脚本和reGeorgSocksProxy.py文件

2. Proxifier

   Proxifier能让原本不支持代理的程序走我们设置的代理，类似于一个socks客户端。

   下载地址：https://www.proxifier.com

   有MacOS版本和Windows版，注册码网上搜索。

## 0x03 使用

#### **配置 reGeorg**

1. 根据环境将tunnel.xxx上传到靶机的Web目录中，tunnel.xxx可解析后访问url检查下是否正常。(一般情况下将原本的文件改成个隐蔽的名字防止被发现)

   - 访问正常，显示Georg says, ‘All seems fine’。
   - 访问异常，根据报错信息检查，一般情况下是敏感函数被拦截了。
2. 本地执行命令
```
python reGeorgSocksProxy.py -p xxxx -u http://:www.xxx.com/xxx/xxx/tunnel.xxx
-p 指定本地代理的端口
-u 上传脚本的url
```

正常的截图如下：

![001](/img/reGeorg_Proxifier/001.png)


#### **配置Proxifier**

1. 在代理选项卡中添加本地代理的端口，我这里设置的是8088

![002](/img/reGeorg_Proxifier/002.png)

2. 配置需要代理的软件

以firefox为例，先去掉本地直连的勾选

![003](/img/reGeorg_Proxifier/003.png)

添加firefox软件走代理

![004](/img/reGeorg_Proxifier/004.png)

到这一步就完成了，我们本机能访问内网了。