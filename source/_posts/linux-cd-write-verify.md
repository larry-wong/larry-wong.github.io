---
title: Linux下CD盘的刻录以及验证
id: 224
categories:
  - Linux
date: 2014-05-14 23:11:19
tags:
---

在Linux下刻录了一张系统CD盘，记录过程：
系统：Fedora 19

下载完iso文件后先用md5sum验证
```
[larry@Larry-Fedora19 ~]$ md5sum debian-7.5.0-amd64-lxde-CD-1.iso 
b7f0b3c860f845768fbc7d2d8c923ad8  debian-7.5.0-amd64-lxde-CD-1.iso
```
使用wodim烧录
```
[larry@Larry-Fedora19 ~]$ wodim -v dev=/dev/cdrom debian-7.5.0-amd64-lxde-CD-1.iso 
wodim: No write mode specified.
wodim: Assuming -tao mode.
wodim: Future versions of wodim may have different drive dependent defaults.
TOC Type: 1 = CD-ROM
wodim: Operation not permitted. Warning: Cannot raise RLIMIT_MEMLOCK limits.
scsidev: '/dev/cdrom'
devname: '/dev/cdrom'
scsibus: -2 target: -2 lun: -2
Linux sg driver version: 3.5.27
Wodim version: 1.1.11
SCSI buffer size: 64512
Device type    : Removable CD-ROM
Version        : 5
Response Format: 2
Capabilities   : 
Vendor_info    : 'ASUS    '
Identification : 'DRW-24D3ST      '
Revision       : '1.01'
Device seems to be: Generic mmc2 DVD-R/DVD-RW.
Current: 0x0009 (CD-R)
Profile: 0x0015 (DVD-R/DL sequential recording) 
Profile: 0x0016 (DVD-R/DL layer jump recording) 
Profile: 0x0018 (Reserved/Unknown) 
Profile: 0x002B (DVD+R/DL) 
Profile: 0x001B (DVD+R) 
Profile: 0x001A (DVD+RW) 
Profile: 0x0014 (DVD-RW sequential recording) 
Profile: 0x0013 (DVD-RW restricted overwrite) 
Profile: 0x0012 (DVD-RAM) 
Profile: 0x0011 (DVD-R sequential recording) 
Profile: 0x0010 (DVD-ROM) 
Profile: 0x000A (CD-RW) 
Profile: 0x0009 (CD-R) (current)
Profile: 0x0008 (CD-ROM) 
Profile: 0x0002 (Removable disk) 
Using generic SCSI-3/mmc   CD-R/CD-RW driver (mmc_cdr).
Driver flags   : MMC-3 SWABAUDIO BURNFREE 
Supported modes: TAO PACKET SAO SAO/R96P SAO/R96R RAW/R16 RAW/R96P RAW/R96R
Drive buf size : 618464 = 603 KB
Beginning DMA speed test. Set CDR_NODMATEST environment variable if device
communication breaks or freezes immediately after that.
FIFO size      : 4194304 = 4096 KB
Track 01: data   648 MB        
Total size:      744 MB (73:43.70) = 331778 sectors
Lout start:      744 MB (73:45/53) = 331778 sectors
Current Secsize: 2048
ATIP info from disk:
  Indicated writing power: 4
  Is not unrestricted
  Is not erasable
  Disk sub type: Medium Type A, low Beta category (A-) (2)
  ATIP start of lead in:  -12508 (97:15/17)
  ATIP start of lead out: 359845 (79:59/70)
Disk type:    Short strategy type (Phthalocyanine or similar)
Manuf. index: 22
Manufacturer: Ritek Co.
Blocks total: 359845 Blocks current: 359845 Blocks remaining: 28067
Speed set to 8467 KB/s
Starting to write CD/DVD at speed  48.0 in real TAO mode for single session.
Last chance to quit, starting real write in    0 seconds. Operation starts.
Waiting for reader process to fill input buffer ... input buffer ready.
Performing OPC...
Starting new track at sector: 0
Track 01:  648 of  648 MB written (fifo 100%) [buf  99%]  47.2x.
Track 01: Total bytes read/written: 679477248/679477248 (331776 sectors).
Writing  time:  146.557s
Average write speed  32.7x.
Min drive buffer fill was 97%
Fixating...
Fixating time:   11.459s
BURN-Free was never needed.
wodim: fifo had 10703 puts and 10703 gets.
wodim: fifo was 0 times empty and 6442 times full, min fill was 92%.
[larry@Larry-Fedora19 ~]$
```

将光盘中的内容复制出来进行验证（强迫症）
本打算用dd复制，但总是失败，结果如下
```
[larry@Larry-Fedora19 ~]$ dd if=/dev/cdrom of=temp.iso
4+0 records in
4+0 records out
2048 bytes (2.0 kB) copied, 1.35707 s, 1.5 kB/s
[larry@Larry-Fedora19 ~]$</pre>
使用readcd复制
<pre>[larry@Larry-Fedora19 ~]$ umount /dev/cdrom
[larry@Larry-Fedora19 ~]$ readcd -v dev=/dev/cdrom -f temp.iso
scsidev: '/dev/cdrom'
devname: '/dev/cdrom'
scsibus: -2 target: -2 lun: -2
Linux sg driver version: 3.5.27
Read  speed:  8467 kB/s (CD  48x, DVD  6x).
Write speed:  8467 kB/s (CD  48x, DVD  6x).
Capacity: 331778 Blocks = 663556 kBytes = 648 MBytes = 679 prMB
Sectorsize: 2048 Bytes
Copy from SCSI (2,0,0) disk to file 'temp.iso'
end:    331778
Errno: 5 (Input/output error), read_g1 scsi sendcmd: no error
CDB:  28 00 00 05 10 00 00 00 02 00
status: 0x2 (CHECK CONDITION)
Sense Bytes: 70 00 05 00 00 00 00 0A 00 00 00 00 26 00 00 00
Sense Key: 0x5 Illegal Request, Segment 0
Sense Code: 0x26 Qual 0x00 (invalid field in parameter list) Fru 0x0
Sense flags: Blk 0 (not valid) 
cmd finished after 0.128s timeout 40s
readcd: Input/output error. Cannot read source disk
readcd: Retrying from sector 331776.
.~~-~~~+~~~-~~~+~~~-~~~+~~~-~~~+~~~-~~~+~~~-~~~+~~~-~~~+~~~-~~~+~~~-~~~+~~~-~~~+~~~-~~~+~~~-~~~+~~~-~~~+~~~-~~~+~~~-~~~
readcd: Input/output error. Error on sector 331776 not corrected. Total of 1 errors.
Errno: 5 (Input/output error), read_g1 scsi sendcmd: no error
CDB:  28 00 00 05 10 00 00 00 01 00
status: 0x2 (CHECK CONDITION)
Sense Bytes: 70 00 05 00 00 00 00 0A 00 00 00 00 26 00 00 00
Sense Key: 0x5 Illegal Request, Segment 0
Sense Code: 0x26 Qual 0x00 (invalid field in parameter list) Fru 0x0
Sense flags: Blk 0 (not valid) 
cmd finished after 1.475s timeout 40s

Time total: 375.180sec
Read 663552.00 kB at 1768.6 kB/sec.
Max corected retry count was 0 (limited to 128).
The following 1 sector(s) could not be read correctly:
331776
[larry@Larry-Fedora19 ~]$
```
这里便卡壳了，第331776个扇区读不出来，以为是CD盘的问题，重新刻录了一张，结果还是一样。后来猜想是不是331776已经到结尾了，于是
```
[larry@Larry-Fedora19 ~]$ md5sum temp.iso 
b7f0b3c860f845768fbc7d2d8c923ad8  temp.iso
```
嘿嘿，跟第一步一模一样的:D，浪费一张CD。

**注：**
- 将光盘弹出再插入之后，dd命令生效
```
[larry@Larry-Fedora19 ~]$ dd if=/dev/cdrom of=temp2.iso
dd: error reading ‘/dev/cdrom’: Input/output error
1327104+0 records in
1327104+0 records out
679477248 bytes (679 MB) copied, 146.774 s, 4.6 MB/s
[larry@Larry-Fedora19 ~]$ md5sum temp2.iso 
b7f0b3c860f845768fbc7d2d8c923ad8  temp2.iso
```
