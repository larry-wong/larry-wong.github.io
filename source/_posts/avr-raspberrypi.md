---
title: 基于Raspberry Pi的AVR开发环境搭建以及一个简单走马灯的实现
id: 85
categories:
  - AVR
  - Linux
date: 2013-05-16 22:34:22
tags:
---

想学avr，又担心把主板烧坏，于是入手一块Raspberry Pi用来专门烧写，很久之前的事了，今天才有空实现了出来。
1. Raspberry Pi的系统，我烧的官方的Raspbian “wheezy”， 本打算使用Arch Linux ARM的，不料无法启动，于是放弃了。
2. Raspberry Pi初次开机的设置，大家都懂的，别忘了开启ssh服务。
3. Raspberry Pi的网络：我淘宝了块usb的无线网卡，这样可以使它更自由些。说下配置方法：
    备份/etc/network/interfaces并修改，我的文件内容如下：
```
pi@raspberrypi /etc/network $ cat interfaces
auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0
iface wlan0 inet static
wpa-ssid "MIMI"
wpa-psk "xiaoyanyan"
address 192.168.2.11
netmask 255.255.255.0
gateway 192.168.2.1
network 192.168.2.1
```
    你需要改下wpa-ssid/wifi名字 和wpa-psk/wifi密码。如果不想设置静态ip，则把static改成dhcp，下面的address, netmask, gateway和network就不需要了。
之后重启网络服务：
```
sudo /etc/init.d/networking restart
```
4. 网络设置好了，可以拔掉显示器键盘等外设，用ssh操作了。为了习惯，可以安装一些vim之类的常用软件。接下来安装avr的开发环境：
```
sudo apt-get install avrdude gcc-avr gdb-avr avr-libc binutils-avr
```
5. nfs的挂载
    我的想法是用Raspberry Pi编译和烧写avr，代码并不放在Raspberry Pi上。于是要在你的开发机上搭建nfs服务器，可以参考我的另一篇文章
[Linux Mac Windows之间的文件共享（NFS &amp; Samba）](/2013/05/10/linux-mac-windows-nfs-samba/ "Linux Mac Windows之间的文件共享（NFS &amp; Samba）")
我在/bin下创建了一个权限为755的mount_nfs文件，内容为“mount 192.168.2.10:/home/larry/workspace/avr /home/pi/avr”，然后在/etc/rc.local文件加入了“mount_nfs”。这样每次启动Raspberry Pi的时候就会自动挂载nfs，即使你的nfs服务器是在Raspberry Pi之后启动的，那么只要在Raspberry Pi执行sudo mount_nfs就可以了。

6. Raspberry Pi里没有ll命令，这可能会有些不习惯，于是我创建了文件~/bin/ll，修改权限为755，内容为“ls -l $@”，然后在～/.bashrc文件最后加一行“export $PATH:$HOME/bin”。或者直接在.bashrc里定义alias ll='ls -l'，这样便可以随意使用ll命令了。

7. 至此Raspberry Pi的环境已经搭载完成了，接下来写一个简单的走马灯程序。main.c：
```
#include <avr/io.h>
#include <util/delay.h>
int main()
{
    DDRB = 0xff;
    int j, i;
    while(1)
    {
        for(j = 0; j < 3; j++)
        {
            PORTB = 0x01;
            for(i = 0; i < 8; i++)
            {
                _delay_ms(20);
                PORTB = PORTB << 1;
            }
        }
        for(j = 0; j < 3; j++)
        {
            PORTB = 0x00;
            _delay_ms(20);
            PORTB = 0xff;
            _delay_ms(20);
        }
    }
}
```
    程序很简单，在PORTB的8个口上各串联一个二极管和电阻，二极管的阴级接地。应该会看到二极管会轮流闪3轮然后一起闪3轮，然后重复。
然后创建一个makefile（参照[第一个AVR程序，在linux下搭建avr的开发环境](http://chou.it/2013/03/set-up-avr-on-linux/ "第一个AVR程序，在linux下搭建avr的开发环境")）：
```
MCU=atmega16a
F_CPU=12000000
SRC=main.c
OUT=main
HEX_FLASH=$(OUT).hex

$(HEX_FLASH): $(OUT)
        avr-objcopy -O ihex -R .eeprom -R .fuse -R .lock -R .signature $(OUT) $(HEX_FLASH)

$(OUT) : main.o
        avr-gcc -mmcu=$(MCU) -o $(OUT) *.o
main.o: $(SRC)
        avr-gcc -DF_CPU=$(F_CPU) -funsigned-char -funsigned-bitfields -Os -fpack-struct -fshort-enums -g2 -c -std=gnu99 -mmcu=$(MCU) $(SRC)

clean:
        rm -f *.o $(HEX_FLASH) $(OUT)

burn: $(HEX_FLASH)
        avrdude -p m16 -c usbasp -e -U flash:w:$(HEX_FLASH)
```
    连接好下载器以及开发板之后，make，make burn就将程序烧制单片机上了，可以检验二极管的效果。

8. 焊接万用板，如果你愿意的话。第一次使用烙铁，焊得很丑：
![](//res.cloudinary.com/larry/image/upload/c_scale,w_300/a_180/v1469583825/avr_raspberrypi_1_qqn0a6.jpg) ![](//res.cloudinary.com/larry/image/upload/c_scale,h_300/a_90/v1469547293/avr_raspberrypi_2_cwbanf.jpg)

附上一张全家福（Raspberry Pi， avr最小板， 我焊的走马灯）：
![](//res.cloudinary.com/larry/image/upload/q_40/v1469583102/avr_raspberrypi_3_i4hp9k.jpg)

记录下遇到的一些问题：
- Raspberry Pi无论烧入什么系统都无法正常启动：换了一张sd卡得到解决，虽然ARM的系统还是不能开机，但是至少debain的可以了。
- 当Raspberry Pi的无线网卡工作后，接入Raspberry Pi的键盘无效：换了一个电源适配器得到解决，原来的是买小音响送的，估计电流不足。
- 无法烧录代码到avr，说usbasp找不到：原来贪便宜，买的avr下载器不是通用的，只能使用于特一烧录软件，重新买了个下载器，问题得到解决。
- 开始用面包版测试走马灯的时候，有几个口不亮，以为IO口坏了，原来那几个IO口默认已被他用，需要设置。
