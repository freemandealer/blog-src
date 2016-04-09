title: 为Android手机编译和安装内核(HTC G14XE/G18)
date: 2014-04-28 15:27:10
tags: Note
---
该文章为本人编译和安装内核过程的记录，目的是作为学习笔记供以后参考．虽然已经努力将其向教程靠拢，但精力有限，主要还是集中注意力记录过程而忽略理论讲解，并且不保证按照此文步骤去做就一定可以成功．

<!--more-->

##思路##
下载CM的ROM，下载系统对应的内核源代码，编译源代码得到内核镜像，将之替换掉ROM中的内核，刷进手机．

##获取原料##
不是所有的android手机都可以给它编译和安装内核，因为不是所有的手机都被人破解，或是开放源内核代码．尽量下载配套的Android系统(ROM或源代码)和内核源代码.

在[CM下载页](http://download.cyanogenmod.org/)上找到手机支持的ROM并下载．

在GitHub上找到系统对应的内核--[HTC_msm8660](https://github.com/CyanogenMod/android_kernel_htc_msm8660)

##编译##
具体做法见我前面的文章**为cubieboard编译内核**.只不过用上次的那个交叉编译器编译产生了错误，换成了Android4.3源代码`prebuilts`目录下的编译器就好了．然后make:

    PATH=/home/freeman/code/aosp43/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin:$PATH
    make ARCH=arm CROSS_COMPILE=arm-eabi- pyramid_defconfig
    make menuconfig #自定义一些配置,这里新内核wifi打不开，就胡乱打开了一些网络设备的开关
    make -j4
得到zImage.

##制作ROM##
ROM的制作过程可以参看官方给出的Android源代码(非内核源代码)下`system/core/mkboootimg/`下的源代码．理论上可以参考之自己写一个解包／打包器．因为官方的工具使用起来比较复杂，这里我直接使用了Windows下有人写好的`bootimg.exe`工具完成实验(利用wine)．下文中的`boot.img`是官方的ROM提取到的．

###解包###

     wine bootimg.exe  --unpack-bootimg boot.img
执行完后有如下信息需要注意，它们说明了boot.img的组织方式的一些参数，根据不同的机型，参数可能发生局部变化．重新打包时我们需要提供这些参数，具体见下面的**重新打包**．

    base=0x48000000
    page_size=2048
    name=""
    cmdline="console=ttyHSL0 androidboot.hardware=pyramid no_console_suspend=1"
    padding_size=2048
    
执行完命令同时得到`kernel`和`ramdisk.gz`两个文件．修改上一步编译得到的内核zImage名字为kernel替换官方的kernel.

###重新打包###
仔细体会上面参数的使用

    wine bootimg --repack-bootimg  0x48000000 "console=ttyHSL0 androidboot.hardware=pyramid no_console_suspend=1" 2048 2048
    
这样我们得到了含有自己编译内核的boot.img．

##刷ROM##
采用各种办法，将boot.img刷进手机．网上教程一大堆，这里就不详细描述了.有图有真相：

![htc_g18编译安装内核测试](/img/htc_g18_kernel.jpg)

##鸣谢##
王雷(雷妈)提供了如此优秀的实验手机．

[徐凌云(maxwellxxx)](https://maxwellxxx.github.io)提供了雄厚的技术支持．


