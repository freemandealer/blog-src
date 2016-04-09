title: 解决编译安装内核后Wifi不能用的问题
date: 2014-04-30 14:41:15
tags: Note
---
刷完内核，wifi始终打不开．一定是驱动问题．

<!--more-->

首先想到的是编译内核时没有打开选项．从原来可以使用wifi的系统中提取config文件,和目前config文件进行diff,发现两者有些许差距但可以判断wifi失效的原因不在此．

打开可以使用wifi的ROM,可以看到`system/lib/modules/bcmdhd.ko`文件．直觉表明这个应该就是wifi模块了.Android诡异的机制把驱动放在系统里而非内核里！而且linux内核先天就对模块的版本号比较挑剔，一定要是和内核版本一样的驱动模块中才能被加载．由于在刷内核时小小地升了个级，对应原先的内核的驱动模块不能被加载了，所以出现wifi不能的情况．

原因分析到这里，着手解决问题．好在makefile里有这个选项，省的我单独编译了．`DeviceDrivers->Network device support->Wireless LAN->Broadcom 4329/30 wireless cards support`．统一起见，把下面的名字也改称`bcmdhd.bin`.

编译好模块，把它放进原来ROM包里，替换`system/lib/modules/bcmdhd.ko`．顺便又把内核编译了一遍，使用上篇文章的方法，用bootimg工具将内核加进去．最后重新压缩成ZIP，刷机．

原不指望意识流也可以一次性成功，但是事实上wifi被点亮了，激动人心!