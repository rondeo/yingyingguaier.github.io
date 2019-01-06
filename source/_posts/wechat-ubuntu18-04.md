---
title: wechat-ubuntu18.04
date: 2018-11-03 17:22:01
tags: [疑难杂症]
categories: ubuntu
---

# 安装wechat(ubuntu18.04)

---

## 0x00

之前在`https://github.com/geeeeeeeeek/electronic-wechat`下载使用的微信会有一些问题,退出后就无法唤醒，但是后台还有进程，只能使用kill命令杀死。决定重新安装下wechat。

## 0x01

`https://github.com/kooritea/electronic-wechat`

```
cd ~/Downloads/
git clone https://github.com/kooritea/electronic-wechat.git
# 进入仓库
cd electronic-wechat
# 安装依赖, 运行应用
npm install && npm start
# 打包linux应用 这里速度很慢,尝试了好几次
npm run build:linux
```

上述步骤操作完成后会在`~/Downloads/electronic-wechat`生产一个`dist`的目录,里面有打包好的文件`electronic-wechat-linux-x64`

![001](/img/ubuntu/wechat-install/001.png)

双击该文件就能打开wechat,但是这样不方便,每次都要到该目录,下面添加个快捷方式。

## 0x02

```
# 拷贝到opt目录
sudo cp -r ~/Downloads/electronic-wechat/dist/electronic-wechat-linux-x64 /opt/wechat
cd /opt/wechat/electronic-wechat-linux-x64/
# 给wechat配个图标
sudo wget https://raw.githubusercontent.com/geeeeeeeeek/electronic-wechat/master/assets/icon.png
# 确保electronic-wechat有执行权限
```

到/usr/share/applications目录下创建启动快捷方式

```
cd /usr/share/applications
sudo vim wechat.desktop
```

编辑内容

```
[Desktop Entry]
Name=wechat
Type=Application
Exec=/opt/wechat/electronic-wechat-linux-x64/electronic-wechat
Icon=/opt/wechat/electronic-wechat-linux-x64/icon.png
Terminal=false
```

Exec对应要执行的程序,Icon对应图标。

![002](/img/ubuntu/wechat-install/002.png)

已经微信了,添加到收藏夹即可。





