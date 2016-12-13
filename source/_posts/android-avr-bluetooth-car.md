---
title: 基于AVR和Android的蓝牙遥控小车
id: 181
categories:
  - Android
  - AVR
date: 2014-05-12 10:45:52
tags:
---

上一篇基于Arduino制作的小车比较入门，只需像积木那样把模块拼起来，然而，它却开启了我的嵌入式之门。继那之后，又实现了一款基于AVR的蓝牙小车。也是去年的事了，现在才记录。

与上一篇[基于Arduino和Android的蓝牙遥控小车](/2014/05/03/android-arduino-bluetooth-car/ "基于Arduino和Android的蓝牙遥控小车")对比：
- 舍弃昂贵的Arduino，采用性价比更高的AVR ATmega16A做MCU。
- 舍弃昂贵的电机驱动板，使用桥式电路代替。
- 舍弃比较鸡肋的红外避障功能。
- 保留原有的蓝牙接收模块。
- 改进Android的用于界面：端增加蓝牙设备扫描选择界面以及操作杆轨道等。

小车电路图（图中电阻均为102欧姆，NPN均为D882，PNP均为B772）：
![](//res.cloudinary.com/larry/image/upload/v1469544916/android_avr_gbiyvs.png)

小车原理：
> 通过2组桥式电路分别控制驱动电机和转向电机。AVR的PB0-PB3控制驱动电机，PB4-PB7控制转向电机。三极管起开关作用。

> 假设与PBx连接的三极管为Sx，即PB0连接的为S0, PB1连接的为S1,……

> 当PB0，PB1输出低电平，PB2,PB3输出高电平时：
> S0为NPN，根据NPN型三极管的特性，基极为低电平时不导通，同理，S1不导通。
> S2为PNP，根据PNP型三极管的特性，基极为高电平时不导通，同理，S3不导通。
> 此时，电机M处于断路状态，小车静止。

> 当PB0,PB3输入高电平，PB1,PB2输入低电平时：
> S0导通，S1不导通，S2导通，S3不导通，电机正转。

> 当PB0,PB3输入低电平，PB1,PB2输入高电平时：
> S0不导通，S1导通，S2不导通，S3导通，电机反转。

> 转向电机同理。

AVR部分代码：
```
void init(void) {
    DDRB = 0xff; //初始化PB0-PB7为输出

    UBRRL = 51; //根据出厂默认时钟频率为1MHz，设置UBRR以使串口USART波特率为2400bps（与蓝牙接收模块一致）
    UCSRB = (1 << RXEN) | (1 << TXEN); //初始化USART的接收使能和发送使能
    UCSRC = (1 << URSEL) | (3 << UCSZ0); //设置帧长度为8位
    reset();
}

int main() {
    init();
    while(1) {
	while(!(UCSRA &amp; (1 << RXC))); //监测缓冲器有未读的数据
	execute_command(UDR);
    }
}
```

Android端的代码不贴了，贴几张界面截图：
![](//res.cloudinary.com/larry/image/upload/c_scale,w_280/v1469545396/android_avr_ui_1_zfrtl2.png)    <span style='display: inline-block;'> ![](//res.cloudinary.com/larry/image/upload/c_scale,w_420/v1469545398/android_avr_ui_2_q3cojt.png)<br>
![](//res.cloudinary.com/larry/image/upload/c_scale,w_420/v1469545412/android_avr_ui_3_uldty8.png)</span>
Android端实现的功能比较多，一开始设计时有PWM变速、蜂鸣器、刹车、LED灯、激光射线、重力感应以及一个RGB三通道均可PWM调色的变色灯。
这部分设计的Android和AVR代码均已完成，但在焊万用板时出现问题，故只做了一个最简单的版本（前进，后退，左转，右转，停止），如有兴趣，参考svn:
svn://larry-wang.com/android/remote_car_v2 (guest/guest)

最后贴下成品图：
![](//res.cloudinary.com/larry/image/upload/q_40/v1469545431/android_avr_ui_car_yhy6xr.jpg)

**注：**
- 桥式电路中，每个三极管应并一个二极管防止倒流，这里偷懒没有使用。
- AVR与蓝牙接收模块共电源，两个电机共电源，两个电源共地。
- svn中car_v1为简单版本的avr代码，car_v2为复杂版本的avr代码，部分复杂版本Android代码被注释。
- 电路图通过kicad绘制。
- Android界面通过Screenshot Easy截取。
