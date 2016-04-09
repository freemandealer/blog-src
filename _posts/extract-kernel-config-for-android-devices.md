title: 提取Android手机的内核配置文件
date: 2014-04-27 20:48:05
tags: Note
---
本文介绍几种从网络上搜集来的提取Android手机内核配置文件的方法．这是为真机编译内核的一个重要基础．

<!--more-->

##方法一 从当前运行内核中提取配置文件##
使用adb shell连接手机,从手机运行的系统中提取`/proc/config.gz`．

使用adb shell时不太顺利，显示错误：

    freeman@freemanlaptop:~$ adb shell
    error: insufficient permissions for device
    freeman@freemanlaptop:~$ adb devices
    List of devices attached 
    ????????????	no permissions
    
采用下面办法解决：

使用root权限执行以下命令

    adb kill-server
    adb start-server
    exit
可惜实验用机中并没有找到config.gz :-(

##方法二 从现有内核镜像中提取配置文件##
可以通过这个脚本获取已经编译好内核的config文件`/usr/src/linux/scripts/extract-ikconfig`.

	/usr/src/linux/scripts/extract-ikconfig image > my_config
局限性是被提取的内核本身编译时开启`CONFIG_IKCONFIG`选项．另外，如果内核被压缩过也会出现错误．

##方法三 手动配置或默认配置##
`make menuconfig`手动配置，需要经验，难度较大:S

顺便打开OTG功能吧:在`Device Drivers->USB support`里打开OTG相关选项．

或者采用默认配置:

    make config_file_name 
    
