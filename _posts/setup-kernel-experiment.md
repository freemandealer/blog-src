title: Android真机内核实验环境搭建的一种简便方法(俗称:单刷内核)
date: 2014-07-05 18:35:27
tags: Note
---
在[前面的文章](http://freemandealer.github.io/2014/04/28/compile_and_install_kernel_for_android_devices/)中提出了如何在真机中安装自己编译内核的方法：编译内核得到zImage，再用Bootimg打包得到新的Boot.img，再zip打包成ROM最后刷机的．该方法是我和[徐凌云(maxwellxxx)](https://maxwellxxx.github.io)摸索出来的土路子，通过它我们熟悉了Android系统的结构．不过操作十分繁琐，不能满足高效实验的需要．另一方面该方法的弊端在于刷机时替换了整个系统，丢失了大量系统数据．今天发现一个新的更简便的实验方案，主要是结合fastboot工具单刷内核．（之前徐凌云就一直跟我提，我也没好好试试．所以真的需要经常接受新事物 :D）

<!--more-->

###安装测试fastboot###

    sudo apt-get install android-tools-fastboot
    
连接手机，并以bootloader方式重启手机，在手机里选择fastboot．
使用命令：

    sudo fastboot devices
   
测试fastboot．如果一直显示`waiting for devices`那么采用
[这篇博文](http://limssb.blog.163.com/blog/static/1473043720123193223292/)里提到的方式解决．
注意fastboot的很多操作需要管理员权限．

###刷入内核###
按照[为Android手机编译和安装内核(HTC G14XE/G18)](http://freemandealer.github.io/2014/04/28/compile_and_install_kernel_for_android_devices/)一文中的方法编译好内核得到zImage. 进入zImage所在的目录(kernel_src/arch/arm/boot/)，执行命令：

    sudo fastboot boot zImage
    
接着手机会自动重启，开机后可以发现内核已更新．

###总结###
Android系统由内核和系统两部分组成．采用单刷内核的方式，保留了系统中的所安装程序和用户数据．可以认为系统并不知道自己的核心被替换掉了．在这个过程中，原料是编译直接得到的zImage，不用进行其它处理，因此过程简洁高效．这样对内核代码进行修改，编译，通过fastboot刷入内核，启动手机便能看到效果．

当然如果实验的部分可以以模块方式编译，动态加载，那么就不用大动干戈编译替换整个内核了．