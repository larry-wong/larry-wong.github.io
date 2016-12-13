---
title: 基于Android的网络唤醒
id: 8
categories:
  - Android
date: 2013-01-16 23:12:41
tags:
---

所谓网络唤醒（Wake On Lan），即通过给网卡发送魔术包（Magic Package）来实现远程开机。此功能需要主板和网卡的支持，在BIOS可以查看并开启此功能。

第一次看到此项技术的时候，心里很兴奋，因为我一直在想着用手机远程打开家里的电脑，然后远程登录进去操作，虽然不知道这样有什么实用性，但我觉得很“酷”。为实现远程开机，我写了一个android小应用用来发送魔法包，先贴2张截图：
![](//res.cloudinary.com/larry/image/upload/c_scale,w_300/v1469584329/wol_android_ui_1_jljspw.png)  ![](//res.cloudinary.com/larry/image/upload/c_scale,w_300/v1469584332/wol_android_ui_2_vvyysf.png)
界面很简洁，甚至都是Android原生组件，但我就是喜欢实用简单的东西。程序核心代码：
```
private void sendMagicPackage(String host, String mac, String port)
    {
    try {
    String[] hex = mac.split(":");
    byte[] macBytes = new byte[hex.length];
    byte[] magicPacket = new byte[6 + 16 * macBytes.length];
    for(int i = 0; i < 6; i++)
        magicPacket[i] = (byte)0xff;
    for(int i = 0; i < hex.length; i++)
        macBytes[i] = (byte)Integer.parseInt(hex[i], 16);
    for(int i = 0; i < 16; i++)
        System.arraycopy(macBytes, 0, magicPacket, 6 + i * macBytes.length, macBytes.length);
    DatagramPacket packet;
    packet = new DatagramPacket(magicPacket, magicPacket.length, InetAddress.getByName(host), Integer.parseInt(port));
    DatagramSocket socket = new DatagramSocket();
    socket.send(packet);
    socket.close();
    Toast.makeText(this, R.string.send_succeed, Toast.LENGTH_SHORT).show();
    }catch (Exception e) {
    Toast.makeText(this, R.string.send_fail, Toast.LENGTH_LONG).show();
    Log.e(TAG, e.toString());
    }
}
```
从代码中可以看出，Magic Package的格式为6个字节的"FF"和网卡MAC地址的16次循环，我们只需要把这个数据包发到指定的网卡即可，无论你用什么方式。

写完代码，烧至手机，接下来就是测试了。首先在电脑BIOS中打开WOL功能，然后关机。打开手机程序，Add一个WOL，因为在局域网，在Host处填入192.168.1.255，MAC填入目标机器的MAC地址（例如xx:xx:xx:xx:xx:xx），Port随便填个数吧，这里对端口号没有要求。之后点击新建的WOL，便会弹出发送魔术包的确认框，增加此确认框的初衷是防止误按导致远程机器开机，且这只是一个开机的程序，没办法再通过它关机。确认之后，按照程序的逻辑，她会给192.168.1.255（也就是广播）发送魔术包，局域网内的设备都会接受到此包，然后解析包中的MAC地址，如果跟自身的一样，则开机，否则此包无效。因为你填了xx:xx:xx:xx:xx:xx，所以只有目标机器启动了。为什么不直接发送到机器的固定IP（例如192.168.1.252）呢？因为固定IP是在操作系统里设置的，你关机的时候当然不起作用，路由器才不会知道你是哪个呢。

只在局域网中使用当然还不够我的目的，我想在下班回家的路上掏出手机按一下，家里的机器就会开机，等我到家的时候它已经启动完成在那等着我了。家里的IP地址一般是动态的，要实现此目的，可以申请固定IP，但是成本比较高，我选择的是用免费花生壳做动态域名解析，而且好多路由器都直接支持它。在花生壳官网注册免费的帐号，在路由器的域名解析出填入帐号密码，这样它便会将路由器和你的免费的域名（如xxxx.xicp.net）绑定起来。原理也不复杂，花生壳会定期（或者不定期，具体不太清楚）将你的IP和域名提交至它的域名服务器，这样当你访问xxxx.xicp.net的时候，域名服务器就能解析到你路由器的IP了。有了这一步，接下来的就简单了，我们设定一个端口（例如9999，最好不要为80和8080，因为电信可能会把它禁用了），在路由器里加一个端口映射。我第一个思路是将9999端口映射到192.168.1.255，这样当路由器接受到魔术包之后会广播到局域网的所有机器。但我的路由器好像不支持，不让填255。于是换一种思路，在路由器里添加一条IP-MAC绑定（例如192.168.1.4 - xx:xx:xx:xx:xx:xx），这里的IP跟你的机器的IP没有关系，它只是告诉路由器说把发送到这个IP的包发送至这个MAC地址的机器。接下来在端口映射里设置9999映射到192.168.1.4\. 把逻辑再理一下：我们设置Host为xxxx.xicp.net，MAC为xx:xx:xx:xx:xx:xx，Port为9999，确认发送后，此魔术包通过花生壳的域名解析服务器到达了你家的路由器，路由器一看端口是9999，于是映射到192.168.1.4这个内网IP，再一看符合IP-MAC绑定规则，于是把这个包发给了MAC为xx:xx:xx:xx:xx:xx的这台机器，不管你开机不开机，MAC地址总是不变的。这台机器的网卡接受到包之后一看是个魔术包，MAC地址还是自己的MAC地址，那还等什么，果断开机啊，于是你听到了激动人心的开机声。

最后贴一下apk包：[http://larry-wang.com/downloads/wol.apk](http://larry-wang.com/downloads/wol.apk)
