---
layout: post
comments: true
title: 安卓渗透测试平台Kali(Nethunter)搭建教程
category: 网络安全
keywords: 2020, Android, Kali
---

本文内容主要由三个部分构成，Android 刷 Kali 过程，刷机原理简单探索，Android 版本 Kali 的简单使用

# 本教程所使用的软硬件环境

1 OnePlus x 手机一台<br>
2 Windows 7 64 位系统<br>
3 QDLoader-HS-USB_Driver_64bit_Setup<br>
4 OnePlus_X_14_A.04_151103<br>
5 twrp-3.0.2-0-onyx.img<br>
6 nethunter-generic-armhf-kalifs-full-2020.1.zip<br>
7 platform-tools_r29.0.6-windows

下文以 OPX 指代 OnePlus x， win7 指代 Windows 7 64 位系统，twrp 指代 twrp-3.0.2-0-onyx.img

声明：笔者与本文提到的任何软硬件环境无任何利益关系，且不能保证软件的安全性。仅供学习研究参考。

# 安卓刷机背景介绍

所谓刷机，就是给手机重装系统。把手机当作 PC 电脑，那么手机上的 flash 就类比于硬盘。我们把 Windows/Linux 安装到硬盘，就类似于把 Android 安装到 flash。

比如我们的电脑安装了 Windows 系统，硬盘分为两个分区，分别是 C 盘，D 盘。flash 也类似，分为多个分区（partitions）。把 ROM 的对应文件刷写到对应的分区，就完成刷机过程。

在刷机之前，先了解一下手机的多种状态，就样机 OPX 而言，有 4 种状态，分别是<br>
1 Download 状态（高通 soc 才有，通过 9008 COM 通信）<br>
2 fastboot 状态<br>
3 Recovery 状态<br>
4 Android OS 状态

刷机有线刷和卡刷两类，线刷是指通过 usb 线连接手机和电脑，用工具刷 rom。 而卡刷是指把 rom 放到手机本地存储或者是 SD Card，然后通过 Recovery 进行刷机。

这四种状态从启动到运行系统之间的关系如下<br>
启动 > [Download 状态] > fastboot 状态 > Recovery 状态 = Android OS 状态

下文的环境搭建顺序是 先通过 Download 状态刷机， 然后在 fastboot 状态刷 twrp，接下来在 Recovery 状态刷 kali，最终正常启动系统

# Download 状态刷机

在刷机界常听到一句话是把手机刷成砖，所谓的刷成砖是指 fastboot 状态，Recovery 状态，Android OS 状态全都进不去。这时对于普通用户，只能通过 Download 状态刷机。而这个 Download 状态是高通 soc 独有的功能。

这一步大多数情况是不需要执行的，笔者抱着学习的态度，尝试了一下通过 Download 状态刷机。不感兴趣的读者可以直接跳过本章节。

## 准备工作

1 准备 OPX 手机一台，USB 线一根<br>
2 下载 QDLoader-HS-USB_Driver_64bit_Setup，[下载](https://miui.blog/any-devices/qualcomm-qdloader-hs-usb-driver/)<br>
3 下载 OnePlus_X_14_A.04_151103，[下载 1](https://repairmymobile.in/flash/oneplus-x-flash-file/)，[下载 2](https://firmwarefile.com/oneplus-x)<br>
4 安装 QDLoader-HS-USB_Driver_64bit_Setup，并重启电脑

上面的 Stock Firmware ROM 我没找到官方发布渠道，这两个地址提供的 ROM 我无法保证安全性，刷机请谨慎。

## 开始刷 ROM

步骤 1，连接手机和电脑，按住 OPX 音量+键，并同时用 usb 线连接手机和电脑，直到 win7 设备管理器 端口（COM 和 LPT）中出现 Qualcomm HS-USB QDLoader 9008(COM n)

![](http://kafroc.github.io/assets/img/android-kali/2020-05-13_140218.jpg)

步骤 2，打开 MsmDownloadTool V2.0.2 工具， 这个工具在 OnePlus_X_14_A.04_151103 中自带了，在设备类型中，能看到一个是 Com 设备，说明已经准备好了，此时，点击校验按钮，校验 Stock Firmware ROM 是否无误，显示 OK 则没问题。准备就绪后，点击 start，工具的通信状态会显示“正在下载镜像 system.img”, 等待一段时间（约几分钟），等到工具显示 "正在关机...", 则表明刷机完成，此时按电源键开机，即可进入氧 OS 系统

![](http://kafroc.github.io/assets/img/android-kali/2020-05-13_154742.jpg)

到这里 Download 状态刷机过程就结束了， 这时候，手机 flash 的所有分区信息都被重置刷入 Stock Firmware ROM 的文件，完全和出厂设置是一样的。

# fastboot 状态刷 twrp

一般来说，厂商自带的 recovery 功能都比较少，我们想刷入 twrp，让我们开始吧！

## 准备工作

1 准备 OPX 手机一台，USB 线一根<br>
2 下载 adb，fastboot 安卓调试工具集，[platform-tools_r30.0.1-windows.zip](https://dl.google.com/android/repository/platform-tools_r30.0.1-windows.zip)<br>
3 下载 twrp，[twrp-3.0.2-0-onyx.img](https://dl.twrp.me/onyx/)

## 开始刷 Recovery

步骤 1 关闭手机，按住音量+和开机键，直到手机屏幕显示“Fastboot Mode”

步骤 2 通过 usb 线连接手机和电脑

步骤 3 进入 platform-tools 目录，执行命令 fastboot devices

```
C:\Users\test\Desktop\platform-tools_r29.0.6-windows\platform-tools>fastboot.exe devices
1a7c2bf0        fastboot
```

有上面的打印则表明 fastboot 连接正常

步骤 4 执行命令 fastboot oem device-info, 查看 fastboot 是否锁定，如果锁定，则需要解锁才能输入 twrp

![](http://kafroc.github.io/assets/img/android-kali/2020-05-13_162156.jpg)

上图表明当前为锁定状态(Device unlocked: false)，这时输入命令 fastboot oem unlock 尝试解锁，提示"oem unlock is disabled"。 说明 fastboot 状态下无法解锁，好在查了资料发现进入氧 OS 系统的开发者模式，可以解锁 fastboot，操作如下

1 重新启动设备，进入氧 OS<br>
2 打开 Settings -- About phone -- Build number 点击 7 次，退出 About phone， 进入 Developer options<br>
3 使能 OEM unlocking

上述操作执行完成后，重新进入 fastboot mode， 再次执行 fastboot oem unlock， 此时手机提示解锁会删除用户数据，是否继续，点击 yes，手机会重启进入氧系统，重新进入 fastboot mode，查看锁定信息，发现解锁成功了

![](http://kafroc.github.io/assets/img/android-kali/2020-05-13_163333.jpg)

所有一切都准备就绪了，开始刷如 twrp 吧

我把 twrp-3.0.2-0-onyx.img 放在桌面 c:\Users\test\Desktop\

执行命令 fastboot flash recovery c:\Users\test\Desktop\twrp-3.0.2-0-onyx.img

```
Sending 'recovery' (14758 KB)                      OKAY [  0.583s]
Writing 'recovery'                                 OKAY [  0.277s]
Finished. Total time: 0.873s
```

执行命令 fastboot boot c:\Users\test\Desktop\twrp-3.0.2-0-onyx.img，让手机进入 twrp，点击 Reboot -- Recovery，此时会提示你“Install SuperSU now？”，拖动图标选择安装，因为后续刷 kali 需要 root 权限。手机再次进入 Recovery 界面，fastboot 状态刷 twrp 这个过程就大功告成了。

点击 Reboot -- System 进入系统，配置好 WiFi，此时 消息提示 SuperSU Installer，点击进入，选择 TWRP，等待下载完成，完成后，选择 continue，接下来会启动到 recovery 安装 SuperSU，安装好后会自动重启，进入 OOS（氧 OS）

至此，twrp 已经刷写完成

# Recovery 状态刷 kali

上述工作完成后，就可以刷入 kali 了

## 准备工作

1 准备 OPX 手机一台，USB 线一根<br>
2 下载 nethunter-generic-armhf-kalifs-full-2020.1.zip，[下载 Generic ARMhf](https://www.offensive-security.com/kali-linux-nethunter-download/)

## 开始刷 kali

步骤 1 关闭手机，长按音量-键和开机键，使手机进入 recovery 模式

步骤 2 点击 Advanced -- ADB Sideload， 此时手机等待中

步骤 3 进入 platform-tools 目录，执行命令 adb sideload c:\Users\test\Desktop\nethunter-generic-armhf-kalifs-full-2020.1.zip（我把 nethunter-generic-armhf-kalifs-full-2020.1.zip 放在桌面 c:\Users\test\Desktop\）

![](http://kafroc.github.io/assets/img/android-kali/IMG_20200513_173732.jpg)
步骤 4 等待进度完成 100%

重启时，显示了酷炫的界面
![](http://kafroc.github.io/assets/img/android-kali/IMG_20200513_175956.jpg)

接下来进入系统后，要配置网络，安装 SuperSU

Android 平台 Kali 环境搭建到这里就结束了。接下来探索一下刷机的原理。

# 探索刷机的奥秘

无论是 Download 状态/fastboot 状态/Recovery 状态，我们刷机都是用到一些工具，简单操作即可，黑客精神让我们不能停留在脚本小子的层面。

我们来看看刷机奥秘是什么

首先我们在开发者选项中使能 usb 调试，让 adb 连接手机，进入 platform-tools 目录，执行 adb root，发现执行失败，接着执行 adb shell，此时可以获取一个低级别 shell，通过执行命令 su，让我们获取 root shell

首先我们看看 flash 分区情况

```
root@OnePlus:/dev/block # ls
...
mmcblk0
mmcblk0p1
mmcblk0p10
mmcblk0p11
mmcblk0p12
mmcblk0p13
mmcblk0p14
mmcblk0p15
mmcblk0p16
mmcblk0p17
mmcblk0p18
mmcblk0p19
mmcblk0p2
mmcblk0p20
mmcblk0p21
mmcblk0p22
mmcblk0p23
mmcblk0p24
mmcblk0p25
mmcblk0p26
mmcblk0p27
mmcblk0p28
mmcblk0p29
mmcblk0p3
mmcblk0p4
mmcblk0p5
mmcblk0p6
mmcblk0p7
mmcblk0p8
mmcblk0p9
```

接下来我们看看每个分区对应的名称

```
DDR -> /dev/block/mmcblk0p4
DRIVER -> /dev/block/mmcblk0p22
LOGO -> /dev/block/mmcblk0p21
aboot -> /dev/block/mmcblk0p5
boot -> /dev/block/mmcblk0p7
cache -> /dev/block/mmcblk0p15
config -> /dev/block/mmcblk0p26
dbi -> /dev/block/mmcblk0p3
fsc -> /dev/block/mmcblk0p18
fsg -> /dev/block/mmcblk0p17
grow -> /dev/block/mmcblk0p29
misc -> /dev/block/mmcblk0p20
modem -> /dev/block/mmcblk0p1
modemst1 -> /dev/block/mmcblk0p10
modemst2 -> /dev/block/mmcblk0p11
oppodycnvbk -> /dev/block/mmcblk0p12
oppostanvbk -> /dev/block/mmcblk0p13
pad -> /dev/block/mmcblk0p9
param -> /dev/block/mmcblk0p23
persist -> /dev/block/mmcblk0p14
recovery -> /dev/block/mmcblk0p16
reserve1 -> /dev/block/mmcblk0p24
reserve2 -> /dev/block/mmcblk0p25
rpm -> /dev/block/mmcblk0p6
sbl1 -> /dev/block/mmcblk0p2
ssd -> /dev/block/mmcblk0p19
system -> /dev/block/mmcblk0p27
tz -> /dev/block/mmcblk0p8
userdata -> /dev/block/mmcblk0p28
```

看到这些分区标签是不是很熟悉，这些文件在 Stock Firmware ROM 大多都存在，在用工具校验时，也有这些提示

![](http://kafroc.github.io/assets/img/android-kali/2020-05-13_185501.jpg)

那么刷机的过程，应该就是把对应的文件写入分区的过程，而卡刷，线刷的 ROM 包不一样，也是因为不同刷机模式，对 ROM 保护的文件及内容以及打包格式是不一样的。

以 recovery 分区为例，看看是不是我们猜想的这样，验证过程是：dump 出 recovery 分区，把 dump 出的二进制和 twrp 镜像做比较，我们预期的效果是两个二进制是一样的。

查看 recovery 分区大小

```
root@OnePlus:/dev/block/platform/msm_sdcc.1 # cat /proc/partitions
major minor  #blocks  name

...
 179       16      16384 mmcblk0p16
...
```

每个 block 是 1k，那么 recovery 分区是 16MB 大小，我们把整个 recovery 分区 dump 到本地，再通过 adb 从手机中 pull 出来

```
root@OnePlus:/dev/block/platform/msm_sdcc.1 # dd if=/dev/block/mmcblk0p16  of=/sdcard/recv.dump bs=1024 count=16384
```

```
C:\Users\test\Desktop\platform-tools_r29.0.6-windows\platform-tools>adb.exe pull /sdcard/recv.dump .
/sdcard/recv.dump: 1 file pulled, 0 skipped. 3.9 MB/s (16777216 bytes in 4.062s)
```

twrp-3.0.2-0-onyx.img 大小为 14.4M，共 15112192 个字节，recv.dump 是整个分区大小为 16M，经对比发现 recv.dump 的前 15112192 字节和 twrp-3.0.2-0-onyx.img 是完全一样的。

这就印证前面的猜想，即刷机无论是刷 recovery 还是 Android ROM，都是把对应的二进制写入分区。

接下来我做了一个尝试，把 OnePlus_X_14_A.04_151103 中的 recovery.img 通过 adb 传入手机，在 adb shell 中，通过 dd 把 recovery.img 写入 recovery 分区

```
dd if=/sdcard/recovery.img of=/dev/block/mmcblk0p16
```

关机，进入 recovery 模式，发现 twrp 被覆盖回原厂 recovery 了

我猜想线刷和卡刷的差别在于线刷（Download 模式和 fastboot 模式）的时候没有文件系统概念，只能把文件整个写入分区，只能覆盖原有分区文件。

而卡刷不一样，卡刷是在 recovery 模式，而这个模式是精简版的 linux，recovery 是和 android os 同级的，在这个模式下，一方面可以直接和线刷一样直接写分区，也可以挂载原有 Android 文件系统，对 android 文件系统做增删改，实现升级的目的。

# 使用 Android Kali

在 Recovery 状态刷 kali 过程，其实是一个升级的过程，升级 kali 的 zip 包时，会对 boot img 做一些改动，安装 kali 文件系统，安装一些 apk 包，如 NetHunter.apk, NetHunterStore.apk, NetHunterTerminal.apk 等，我们使用 NetHunter 时，应该是 chroot 到 kali 文件系统，这时候对于操作者来说，就像在 kali 环境一样。

Kali 的教程网络上非常多，本文就不班门弄斧了。

我就做一个简单的演示，用 msf 创建一个 payload，修改本机 mac 地址，然后用 nmap 做一个端口扫描。

先打开 NetHunter
![](http://kafroc.github.io/assets/img/android-kali/IMG_20200513_180129.jpg)

![](http://kafroc.github.io/assets/img/android-kali/IMG_20200513_180146.jpg)

## 用 msf 生成 payload

![](http://kafroc.github.io/assets/img/android-kali/IMG_20200513_180620.jpg)

## 修改 MAC 进行 Nmap 扫描

首先打开 NetHunter 应用，点击 MAC Changer，修改 mac 地址为 aa:bb:cc:dd:ee:ff
![](http://kafroc.github.io/assets/img/android-kali/IMG_20200513_201440.jpg)

接下来打开 NetHunter Terminal 应用，执行命令 nmap -A 192.168.1.6（同一个网段的另一台主机）
![](http://kafroc.github.io/assets/img/android-kali/IMG_20200513_201411.jpg)
在主机抓包发现，手机的 MAC 地址确实变成 aa:bb:cc:dd:ee:ff 了。
![](http://kafroc.github.io/assets/img/android-kali/2020-05-13_201511.jpg)

**由于笔者水平有限，文章难免会有错误的，欢迎读者批评指正。笔者个人邮箱：kafrocyang@gmail.com**

# 参考文献

[1][https://repairmymobile.in/flash/oneplus-x-flash-file/](https://repairmymobile.in/flash/oneplus-x-flash-file/)<br>
[2][https://twrp.me/oneplus/oneplusx.html](https://twrp.me/oneplus/oneplusx.html)<br>
[3][https://forum.xda-developers.com/oneplus-7/oneplus-7--7-pro-cross-device-development/rom-kali-nethunter-oneplus-7-oneplus-7-t3976357](https://forum.xda-developers.com/oneplus-7/oneplus-7--7-pro-cross-device-development/rom-kali-nethunter-oneplus-7-oneplus-7-t3976357)
