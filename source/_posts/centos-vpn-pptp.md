---
title: CentOS下基于pptp的VPN服务搭建
id: 240
categories:
  - Linux
date: 2014-12-26 13:42:06
tags:
---

有幸成功搭建了VPN服务，纪录过程以备忘。

服务器版本：CentOS release 6.5（Final）
内核版本：Linux 2.6.32-431.1.2.0.1.el6.x86\_64
VPS服务商：Digital Ocean

搭建步骤：

**1. 确认内核已经加载MPPE模块**
```
sudo modprobe ppp-compress-18 && echo OK
```
如果打印出来OK则表示已经加载MPPE模块。

**2. 安装ppp和pptpd**
```
sudo yum install ppp pptpd
```

**3. 备份即将修改的配置文件**
```
sudo cp /etc/ppp/options.pptpd{,_back}
sudo cp /etc/pptpd.conf{,_back}
sudo cp /etc/sysctl.conf{,_back}
```

**4. 修改配置文件**
```
sudo vim /etc/ppp/options.pptpd
```
这个文件只改了dns为google的，开始没加，导致VPN连接成功之后不能访问网络，修改后的有效代码如下：
```
name pptpd
refuse-pap
refuse-chap
refuse-mschap
require-mschap-v2
require-mppe-128
ms-dns 8.8.8.8
ms-dns 8.8.4.4
proxyarp
lock
nobsdcomp
novj
novjccomp
nologfd
```

/etc/ppp/chap-secrets 是保存用户账号密码信息的，比较好理解
```
yourusername    pptpd    yourpasswd1    *
```

/etc/pptpd.cofig 反注释下两行就OK
```
localip 192.168.0.1
remoteip 192.168.0.234-238,192.168.2.245
```

**5. 打开内核IP转发功能**
修改 /etc/sysctl.conf, 其中
```
net.ipv4.ip_forward = 0
```
修改为
```
net.ipv4.ip_forward = 1
```
然后执行下列命令使之生效
```
sudo sysctl -p
```

**6. 配置iptables**
```
sudo iptables -A INPUT -p gre -j ACCEPT
sudo iptables -A INPUT -p tcp -m tcp --dport 1723 -j ACCEPT
sudo iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 192.168.0.0/24 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -d 192.168.0.0/24 -i eth0 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
sudo /etc/init.d/iptables save
```

**7. 启动pptpd服务**
```
sudo /etc/init.d/pptpd start
sudo chkconfig pptpd on
```

至此，服务端搭建已经完成，MacOS已经可以成功连接了。

**注：**
- 第一次修改iptables之后没有重启pptpd，导致客户端无法连接。
- 本文参考了[Linux下搭建VPN服务器（CentOS、pptp）](http://www.cnblogs.com/sixiweb/archive/2012/11/20/2778732.html "Linux下搭建VPN服务器（CentOS、pptp）")。
