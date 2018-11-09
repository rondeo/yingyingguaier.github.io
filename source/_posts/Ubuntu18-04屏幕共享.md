---
title: Ubuntu18.04屏幕共享
date: 2018-11-08 22:14:59
tags: [疑难杂症]
categories: ubuntn
---

# Ubuntu18.04屏幕共享

---

![001](/img/ubuntu/ScreenSharing/001.png)

使用`dconf Editor`,没有的就使用`apt-get`安装

修改桌面配置。

![002](/img/ubuntu/ScreenSharing/002.png)

设置`require-encryption`为`false`,`view-only`为`true`.

局域网内的就可以通过ip访问了。