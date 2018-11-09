---
title: seafile个人云盘搭建
date: 2018-11-08 23:14:59
tags: [杂项]
categories: [杂项]
---

# seafile个人云盘搭建

## 0x01 seafile 

`https://github.com/haiwen/seafile`

![001](/img/seafile/001.png)

解决的问题:文件同步

![002](/img/seafile/002.png)

## 0x02 搭建

`https://github.com/helloxz/seafile`

![003](/img/seafile/003.png)

`http://144.202.127.105:8000/d/6aefa7d9d5124d28ba59/#`

![004](/img/seafile/004.png)

![005](/img/seafile/005.png)

![006](/img/seafile/006.png)

![007](/img/seafile/007.png)

![008](/img/seafile/008.png)

![009](/img/seafile/009.png)

## 0x03 升级

`https://github.com/neroxps/seafile-server-upgrade`

![010](/img/seafile/010.png)

1. 下载脚本到 seafile 程序目录(之前安装的Mycloud目录)



   ```
   wget https://raw.githubusercontent.com/neroxps/seafile-server-upgrade/master/seafile_upgrade.sh
   ```

   ![011](/img/seafile/011.png)

2.  使用root权限运行脚本

   ```
   sudo chmod +x seafile_upgrade.sh
   sudo ./seafile_upgrade.sh
   #可能还需要
   ./seafile_upgrade.sh -no_backup
   ```

   ![012](/img/seafile/012.png)

![013](/img/seafile/013.png)

```
#重启服务
./seafile.sh start
./seahub.sh start
```

![014](/img/seafile/014.png)