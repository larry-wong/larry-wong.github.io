---
title: Linux Mac Windows之间的文件共享（NFS & Samba）
id: 48
categories:
  - Linux
date: 2013-05-10 22:33:11
tags:
---

前段时间在Fedora18上搭载了NFS服务器，今天有空记录下。

**1. 在fedora18上安装NFS服务器**
```
sudo yum install rpcbind nfs-utils
```

**2. 编辑/etc/exports文件，添加要共享的文件夹。我打算在局域网共享主目录/home/larry，故我的/etc/exports文件内容如下：**
```
[larry@localhost etc]$ cat exports
/home/larry 192.168.2.8(sync,rw,anonuid=1000,anongid=1000,all_squash,insecure) 192.168.2.*(sync,ro,anonuid=1000,anongid=1000,all_squash,insecure) 192.168.2.11(sync,rw,anonuid=1000,anongid=1000,all_squash,insecure)
```
稍稍解释下：
anonuid/anongid：设定当客户端为匿名登陆时的用户uid/gid。
all_squash：无论客户端是什么用户登陆都视为匿名用户（nobody/nfsnobody）。与anonuid/anongid连用意思就很明确了：无论客户端是谁，都被至为uid和gid都为1000的这个用户，也就是larry。因为/home/larry只有larry才有全部的权限。当然你可以把它权限改成777，只不过我不喜欢这么做。查看自己的uid/gid可以使用id命令：
```
[larry@localhost etc]$ id
uid=1000(larry) gid=1000(larry) groups=1000(larry)
```
我仅对ip为8和11的机器开放了写的权限，其他机器只有读的权限。
insecure：每次连接时分配的端口号>=1024，开始没加这个，导致Mac连接时报错。

**3. 关闭iptables**
我的iptables好像有问题，怎么配都不行，索性关了算了。
```
sudo chkconfig iptables off //永久
sudo service iptables stop //即时
```
我的iptables问题还真大，关闭了都不起作用，即使重启也无效。但我意外发现只要执行“service iptables start”， “service iptables stop”就好了，真是怪异。
于是建立文件/etc/rc.d/rc.local，更改权限为755：
```
sudo touch /etc/rc.d/rc.local
sudo chmod 755 /etc/rc.d/rc.local
```
写入内容：
```
[larry@localhost etc]$ cat /etc/rc.d/rc.local 
#! /bin/bash
service iptables start
service iptables stop
```
使用
```
sudo systemctl status rc-local.service
```
检查rc-local的状态，如果没有active，则
```
sudo systemctl start rc-local.service
```

**4. 启动nfs服务**
```
sudo systemctl enable nfs-server.service //永久
sudo systemctl start nfs-server.service //即时
```
至此，服务器部分算是搞定了。

**5. Mac客户端**
```
mount_nfs 192.168.2.10:/home/larry xxx
```
就把服务器的/home/larry挂载到xxx目录了。

**6. win7客户端**
win7自带了NFS客户端了，只是默认没有开启。控制面板 –> 程序 –> 打开或关闭Windows功能，勾上NFS服务，等进度条走完...
打开cmd，敲：
```
mount \\192.168.2.10\home\larry x:\
```
提示没有权限，猜想以上anonuid/anongid和all_squash对win7无效，于是修改注册表：HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default，添加两项：AnonymousUid，AnonymousGid，值都为3e8（16进制的1000）。怎么把1000转成16进制的？大神教我的方法，在浏览器的console里输入：
```
var a=1000;
a.toString(16)
```
它就会给你答案了...

**7. 乱码**
在win7写入到fedora的文件名中文都是乱码，在fedora：
```
sudo yum install convmv
convmv -f GBK -t UTF-8 --notest -r *
```
这样就会递归地把当前目录下的文件名转化过来，但是到win7下看又是乱码了，这里便搞不定了。

最后我还是使用samba来专门对windows进行共享。

**8. samba服务器的安装**
```
sudo yum install samba
```

**9. 给samba添加用户并设置密码**
```
sudo smbpasswd -a larry
```
我是把我自己加入了samba的用户组，没有另外建用户。不过提示说smbpasswd找不到，原来要先：
```
sudo yum install samba-client
```

**10. 备份并修改samba配置文件**
```
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf_back
```
我把/etc/samba/smb.conf后面的[homes]和[printers]都删掉了，自己加入：
```
[larry]
comment = data
path = /home/larry
guest ok = no
writable = yes
valid users = @larry
```
字面意思很好理解，只允许larry用户访问，密码上面已经设置过了。

**11. 关闭selinux**
备份/etc/sysconfig/selinux并修改其中SELINUX值为disabled，重启生效。

**12. 启动samba**
```
sudo systemctl enable smb.service //永久
sudo systemctl start smb.service //即时
```

**13. windows端**
win+R，输入\\192.168.2.10，会提示输入用户名密码。

**14. Mac端**
Finder -> 前往 -> 连接服务器，输入：smb://192.168.2.10，会提示输入用户名密码。

至此，大功告成！在使用过程中发现，服务器上某些名字怪异的文件通过NFS共享到Mac上显示不出来，通过Samba则没有问题。
