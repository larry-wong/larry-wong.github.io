---
title: Ext4文件系统可用空间消失之谜
categories:
  - Linux
data: 2017-02-26 21:12:35
tags:
---

旧1T硬盘，格式化ext4后用作数据存储。然而df后可用空间仅869.2G：
```
Filesystem      1K-blocks       Used Available      Use%    Mounted on
/dev/sdb1       960380628       77868 911448300     1%      /mount-point
```
1T总空间换算后应为10\*\*12/(2\*\*30) = 931.3G。
实际总大小仅为960380628/(2\*\*20) = 915.9G。
实际可用空间911448300/(2\*\*20) = 869.2G。
空间无缘无故消失了60个G，强迫症患者岂能忍？遂google了一番。
原来ext4默认每16k创建一个inode，每个linode 256字节。磁盘用作多媒体文件存储的，自然不必这么多linode，于是umount后：
```
sudo mkfs.ext4 -T largefile /dev/sdb1
```
再次mount后df：
```
Filesystem      1K-blocks       Used        Available       Use%    Mounted on
/dev/sdb1       975405876       77852       926473564       1%      /mount-point
```
975405876/(2\*\*20) = 930.2G。
926473564/(2\*\*20) = 883.6G。
总大小跟可用空间均提升了14G，linode占了不到1G，可以接受。
但是，可用空间为何这么少呢？应该跟总空间持平才是嘛。
继续google，得知ext4默认保留5%空间，用于root用户维护或者系统日志使用。
鉴于此盘完全用于存储，不必此功能：
```
sudo tune2fs -r 0 /dev/sdb1
```
再次df：
```
Filesystem      1K-blocks       Used        Available       Use%    Mounted on
/dev/sdb1       975405876       77852       975311640       1%      /mount-point
```
可用空间跟总大小几乎持平，完美收工！
