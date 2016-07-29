---
title: 基于Arduino和Android的蓝牙遥控小车
id: 160
categories:
  - Android
  - Arduino
date: 2014-05-03 21:24:18
tags:
---

《007-明日帝国》里的那辆酷车，着实让人心潮澎湃，一直想做一辆手机遥控的小车，直到发现Arduino，梦想得以实现。2年多之前的事了，今天回味一下。
Android做上位机，通过蓝牙给Arduino发送控制指令，实现的功能有：
- Android触摸控制。
- Android重力感应控制。
- 小车自动驾驶（红外线避障碍）。
- 小车左右轮通过PWM变速。

操作流程：
- 给小车通电（电机电源和Arduino电源独立）。
- 确认Android的蓝牙已开启，并且与蓝牙接收模块匹过，打开Android客户端，等待蓝牙连接成功。
- 默认是触控模式，界面上左边操作杆控制小车左轮，右边操作杆控制右轮。操作杆向前推，对应的轮子向前进，推动幅度越大，速度越快。同理，向后推，轮子向后转。
- 点击屏幕上的重力感应开关，进入重力感应模式，手机水平放置时，小车静止。手机向任意角度倾斜，Android会计算出左右轮的方向与速度，发送给Arduino。
- 点击屏幕上的自动驾驶开关，进入红外避障模式，此时小车不响应Android的任何指令（退出红外避障模式指令除外），通过红外避障模块的I/O口检测，Arduino自动控制小车前进、转向、后退以及停止。

各个模块。
Arduino:
![](https://res.cloudinary.com/larry/image/upload/v1469545955/android_arduino_1_p4y3mr.png)
对于Arduino的介绍就不多说了，[官网](http://www.arduino.cc/)资料很详尽。

蓝牙接收模块:
![](https://res.cloudinary.com/larry/image/upload/v1469545944/android_arduino_2_aourtz.png)
这里我只用到了VCC/GND供电以及RXD/TXD与Arduino进行串口通信。

红外探测模块:
![](https://res.cloudinary.com/larry/image/upload/v1469545944/android_arduino_3_qy8rai.png)
此模块由一个主模块（左）和四个子模块（右）构成。每个子模块探测单个方向的障碍物情况，主模块通过4个I/O口与Arduino通信。

电机驱动模块:
![](https://res.cloudinary.com/larry/image/upload/v1469545944/android_arduino_4_o1g8tf.png)
这是一款两路的电机驱动，用来控制小车左右两边的电机。EA和EB接Arduino的PWM口，控制电机转速，IA和IB接普通I/O口，控制电机转向。OUT\_A和OUT\_B分别接小车两边的电机。

电路图：
![](https://res.cloudinary.com/larry/image/upload/v1469545948/android_arduino_5_khsarl.png)

主要原理：
蓝牙接收模块将Android发送的命令通过串口传给Arduino，Arduino对命令解析，发送相应的指令到电机驱动模块。在红外避障模式下，Arduino时刻检测红外模块的4个输出值，一旦某个值有变化（遇到障碍物），则发送相应的指令给电机驱动模块。
定义好Android与Arduino之间通讯的数据格式为一个8位的数字。前2位分表表示左右两边电机的转向，中间3位和最后3位分别表示左右两边电机的转速（0～255）。此外定义2个特殊的命令：进入避障模式（99000001）和离开避障模式（99000002）。
触控模式下，Android通过屏幕上2个操作杆的位置决定以上四个值，重力感应模式下则有手机与水平面的倾斜角度（x，y）决定。
Arduino主循环中读取串口的字节，每八个字节组成一条指令，根据指令控制电机。

具体代码就不贴了，了解原理，代码很简单。
svn://larry-wang.com/android/remote\_car(guest/guset)，包含Android和Arduino代码。

最后贴下成品图：
![](https://res.cloudinary.com/larry/image/upload/q_40/v1469545962/android_arduino_car_y7icxz.png)

**注：**
- 避障模式的转弯逻辑有待改进，探测到前方有障碍物时，目前的逻辑是拐弯，直到前方没有障碍。但由于每方只有一个红外探头，且探测的距离有限，会导致小车只拐弯45度左右，此时还是不能前进的。可以在小车拐弯时，加一个主循环的次数，保证小车拐弯90度了再前进。
- 为防止干扰，Arduino和电机的电源是分开来的，2个电源一定要共地。
- 电路图通过kicad绘制。
