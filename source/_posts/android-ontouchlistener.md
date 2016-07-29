---
title: 关于Android中OnTouchListener多点触摸的初探
id: 250
categories:
  - Android
date: 2015-01-08 17:07:31
tags:
---

最近项目中涉及到Android的多点触摸，借此机会研究了一下，在此纪录，以备忘。
测试机器：Moto Milestone2
测试系统： Android2.2
首先，一般情况下，界面上的不同View是不可以一起按的。在某个View的按住状态下，其他View无法按下去。可能是有实现的办法我没找到，若有知晓，不令赐教。
下面说说在同一个View下触摸的情况。
MotionEvent类中有几个静态常量：
```
MotionEvent.ACTION_DOWN //0x0
MotionEvent.ACTION_POINTER_DOWN //0x5
MotionEvent.ACTION_POINTER_1_DOWN //0x5
MotionEvent.ACTION_POINTER_2_DOWN //0x105
MotionEvent.ACTION_POINTER_3_DOWN //0x205
MotionEvent.ACTION_MOVE //0x2
MotionEvent.ACTION_UP //0x1
MotionEvent.ACTION_POINTER_UP //0x6
MotionEvent.ACTION_POINTER_1_UP //0x6
MotionEvent.ACTION_POINTER_2_UP //0x106
MotionEvent.ACTION_POINTER_3_UP //0x206
MotionEvent.ACTION_POINTER_ID_MASK //0xff00
```

以下是多点测试结果：
- 第一个手指按下时，e.getAction() == 0x0，e.getPointerCount() == 1，触发ACTION_DOWN；
- 此时第二个手指按下，e.getAction() == 0x105，e.getPointerCount() == 2，触发ACTION_POINTER_2_DOWN;
- 松开第二个手指，e.getAction() == 0x106，e.getPointerCount() == 2，触发ACTION_POINTER_2_UP；
- 松开第一个手指，e.getAction() == 0x1，e.getPointerCount() == 1，触发ACTION_UP；

倘若最后2部循序颠倒：
- 先松开第一个手指，e.getAction() == 0x6，e.getPointerCount() == 2，触发ACTION_POINTER_1_UP；
- 松开第二个手指，e.getAction() == 0x1，e.getPointerCount() == 1，触发ACTION_UP；

根据以上的测试结果，可通过
```
e.getAction() == MotionEvent.ACTION_DOWN || (e.getAction() & 0xff) == MotionEvent.ACTION_POINTER_DOWN
```
来判断是按下操作，通过
```
(e.getAction() & MotionEvent.ACTION_POINTER_ID_MASK) >> 8
```
来获取此次按下的pointerId，再通过e.getX(pointerId)和e.getY(pointerId)来获取此次按下的手机的坐标。
ACTION_UP同理。
