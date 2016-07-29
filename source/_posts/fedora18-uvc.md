---
title: Fedora18 USB摄像头驱动（UVC）
id: 114
categories:
  - Linux
date: 2013-05-26 13:29:18
tags:
---

最近入了一块蓝色妖姬的摄像头，内置麦，连上fedora18(LXDE)后两者都不工作。记录一下配置过程。
麦克风比较简单，直接上图：
![](https://res.cloudinary.com/larry/image/upload/v1469546452/fedora18_uvc_rymmyl.png)
注意Buit-in Audio选的是Output而不是Duplex。

视频部分安装UVC驱动：
lsusb找到你的摄像头的Device ID，如果包含在[http://www.ideasonboard.org/uvc/](http://www.ideasonboard.org/uvc/ "http://www.ideasonboard.org/uvc/")的列表内，那么请继续。
```
cd
git clone --depth=1 git://linuxtv.org/media_build.git
cd media_build 
./build
sudo make install
sudo reboot
```
./build的时候可能会提示你安装依赖，安装就是了，而且build的时间比较长，纵使3770k的核，也花了四五分钟。
重启后在cheese和skype中都能看到视频了。
