---
title: 安装Fedora 19之后的一些折腾
id: 127
categories:
  - Linux
date: 2013-07-12 15:52:33
tags:
---

由于系统崩溃，安装了fedora19, 记点笔记，备忘。

**1. compat drivers**
新板子上的网卡fedora默认不能识别，先弄个usb的网卡插着，把compact drivers装好：
```
[larry@Larry-Fedora19 ~]$ cat /proc/version 
Linux version 3.9.9-301.fc19.x86_64 (mockbuild@bkernel02) (gcc version 4.8.1 20130603 (Red Hat 4.8.1-1) (GCC) ) #1 SMP Thu Jul 4 15:10:36 UTC 2013
```
得到内核版本是3.9.9，从 http://drvbp1.linux-foundation.org/~mcgrof/rel-html/backports/ 下载对应自己内核的版本。
安装依赖：
```
sudo yum install kernel-headers.x86_64 gcc
```
解压compat drivers并cd进入：
```
make
make install
```
它会自动修改grub，并添加一项名字带有“with compat-drivers”的默认启动项。而这个启动项我无法启动，修改/boot/grub2/grub.cfg，将之前的启动项设为默认即可（进去也带有网卡驱动）。
整个过程折腾了许久，希望fedora早日集成此驱动。

**2. 添加163和sohu的源**
http://mirrors.163.com/
http://mirrors.sohu.com/

**3. fcitx**
```
sudo yum install fcitx.x86_64 fcitx-pinyin.x86_64
```
刚开始安装的 fcitx-cloudpinyin.x86_64，不起作用！

**4. sudo chkconfig iptables off 依然无效，还是需要：**
```
sudo touch /etc/rc.d/rc.local
sudo chmod 755 /etc/rc.d/rc.local

[larry@localhost etc]$ cat /etc/rc.d/rc.local 
#! /bin/bash
service iptables start
service iptables stop

sudo systemctl start rc-local.service
```

**5. 安装nfs之后sudo systemctl enable nfs-server.service无效，但sudo systemctl start nfs-server.service有效**
在/etc/rc.d/rc.local中添加“systemctl start nfs-server.service”

**6. Gnome还是不习惯，返回LXDE**
```
sudo yum install @lxde-desktop
```

**7. 蓝牙小图标**
```
sudo yum install blueman
```

**8. Eclipse快捷方式**
新建vim /usr/share/applications/eclipse.desktop，内容如下：
```
[Desktop Entry]
Name=Eclipse
Comment=Eclipse
Exec=/opt/eclipse/eclipse
Icon=/opt/eclipse/icon.xpm
Terminal=false
Type=Application
Categories=Application;Development;
```
