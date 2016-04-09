title: 使用KGDB调试Android的Goldfish内核
date: 2014-11-29 19:57:26
tags: Note
---


## 内核调试 ##

<!--more-->

为目标机器的内核打开kgdb及相关调试选项并编译内核。将内核改名'kernel-qemu'拷贝至`out/target/product/generic/`目录。

需要打开的内核选项，选项在 Kernel hacking 里可以找到：

    kgdb 选项 ：
        CONFIG_KGDB=y
        CONFIG_KGDB_SERIAL_CONSOLE=y
        
    为了能在系统运行时中断系统并出发远程 gdb，必须打开内核 Magic Sys-Rq 键选项 ：
        CONFIG_MAGIC_SYSRQ=y
        
    打开内核符号调试：
        CONFIG_DEBUG_INFO=y

启动模拟器：

    emulator -verbose -show-kernel -netfast -sysdir out/target/product/generic/ -sdcard sdcard.img -qemu -gdb tcp::1234,ipv4 -S
        
主机使用的gdb所在目录：

    aosp43/prebuilts/gcc/linux-x86/arm/arm-eabi-4.7/bin/arm-eabi-gdb

使用下面的命令进行调试：

    arm-eabo-gdb vmlinux
        
注意这里的vmlinux 不是 `kernel/arch/arm/boot/compressed/`几兆的那个`vmlinux`，而需要重新编译`make -B vmlinux`得到五、六十兆大小的`vmlinux`。

在gdb中使用下面命令连接目标机器：

    target remote localhost:1234

## 模块调试(Android上无效) ##

模块调试信息不存在于之前的 vmlinux 文件中，需要手动添加：

在目标机器使用下面的命令:

    sudo cat /sys/module/<your-module>/sections/.text

得到模块加载地址<address>。

    add-symbol-file <your-module.ko> <address>
    
在Android调试过程中，'.text'文件内容一直是'0x00000000'，故此方法行不通，只能编译进内核调试代码而不是作为模块调试。在真实机器中，情况则不同。如果真机也是'0'，则需要使用root权限查看'.text'文件。


## 参考 ##
打印调试的方法：[How to get kernel messages from Android](http://bootloader.wikidot.com/linux:android:kmsg)

真机使用USB调试的方法：[Enabling KGDB for Android](http://bootloader.wikidot.com/android:kgdb)


    

