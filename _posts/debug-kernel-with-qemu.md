title: 用QEMU来调试内核 -- 道听途说篇
date: 2015-08-27 09:28:12
tags: Note
---
在KernelNewbies列表上呆了两年了，问过几个问题，回答过几个问题，更多地是看高手对话，偷偷学艺。这几天人们在列表上谈论了如何QEMU调试内核的话题，我自己试了试，发现比用VMware调试强大很多(使用VMware调试内核请看[之前文章](http://freemandealer.github.io/2015/03/18/kernel-debugging/)):

- 简单、轻便，通过命令参数指定内核
- 编译出内核即能调试，不需要应用层
- 能调试诸多硬件平台(ARM/MIPS等等)

下面是我从邮件中整理出的具体步骤，部分亲测。

<!--more-->

## 调试内核
### 编译带调试信息的内核镜像 ###

首先我们得有一个被调试的内核二进制镜像。这个内核镜像必须是打开调试模式编译出来的，即内核配置中：

	CONFIG_DEBUG_INFO=y

### 运行内核 ###
根据邮件原文，接着运行:

	qemu -s -S  -kernel arch/x86/boot/bzImage -append "console=ttyS0" -serial mon:stdio -nographic

或者

	qemu -s -S -smp 1,cores=1  -hda /dev/zero -kernel arch/x86/boot/bzImage

就可以启动内核。但我在操作时，单独一个`qemu`并不是一个程序，而是：

	qemu-aarch64              qemu-mipsn32              qemu-system-i386        
	qemu-aarch64-static       qemu-mipsn32el            qemu-system-lm32        
	qemu-alpha                qemu-mipsn32el-static     qemu-system-m68k        
	qemu-alpha-static         qemu-mipsn32-static       qemu-system-microblaze  
	qemu-arm                  qemu-mips-static          qemu-system-microblazeel
	qemu-armeb                qemu-nbd                  qemu-system-mips        
	qemu-armeb-static         qemu-or32                 qemu-system-mips64      
	qemu-arm-static           qemu-or32-static          qemu-system-mips64el    
	qemu-cris                 qemu-ppc                  qemu-system-mipsel      
	qemu-cris-static          qemu-ppc64                qemu-system-moxie       
	qemu-debootstrap          qemu-ppc64abi32           qemu-system-or32        
	qemu-i386                 qemu-ppc64abi32-static    qemu-system-ppc         
	qemu-i386-static          qemu-ppc64-static         qemu-system-ppc64       
	qemu-img                  qemu-ppc-static           qemu-system-ppcemb      
	qemu-io                   qemu-s390x                qemu-system-s390x       
	qemu-m68k                 qemu-s390x-static         qemu-system-sh4         
	qemu-m68k-static          qemu-sh4                  qemu-system-sh4eb       
	qemu-make-debian-root     qemu-sh4eb                qemu-system-sparc       
	qemu-microblaze           qemu-sh4eb-static         qemu-system-sparc64     
	qemu-microblazeel         qemu-sh4-static           qemu-system-unicore32   
	qemu-microblazeel-static  qemu-sparc                qemu-system-x86_64      
	qemu-microblaze-static    qemu-sparc32plus          qemu-system-xtensa      
	qemu-mips                 qemu-sparc32plus-static   qemu-system-xtensaeb    
	qemu-mips64               qemu-sparc64              qemu-unicore32          
	qemu-mips64el             qemu-sparc64-static       qemu-unicore32-static   
	qemu-mips64el-static      qemu-sparc-static         qemu-x86_64             
	qemu-mips64-static        qemu-system-alpha         qemu-x86_64-static      
	qemu-mipsel               qemu-system-arm                                   
	qemu-mipsel-static        qemu-system-cris                                 

这么多`qemu-xxx`... 其中`qemu-system-xxx`是运行`xxx`硬件平台下整个操作系统的模拟器，比如我有一个x86_64平台的内核，那就使用下面命令运行之：

	qemu-system-x86_64 -s -S -kernel /boot/vmlinuz-3.13.0-58-generic -append "console=ttyS0" -serial mon:stdio -nographic

### 调试！ ###
接着就可以打开GDB进行连接、调试:

	$gdb vmlinux
	(gdb) target remote localhost:1234
	(gdb) b start_kernel
	(gdb) c

## 调试模块
qemu运行参数需要加上`-initrd　<Your Init-RamDisk>`选项，确保模块被安装。如果没有现成的initrd，那就做一个：

	sudo make modules_install
	mkinitrdramfs -o initrd.img -v <your-version>

接着使用下列命令载入模块符号：

	(gdb) add-symbol-file /home/aruna/kmod/misc.ko 0xffffffffa057e000 -s .data　0xffffffffa0580000 -s .bss 0xffffffffa05802c8

后面的那些地址可以通过如下命令查看:

	sudo cat /sys/module/<your-module>/sections/.text
	sudo cat /sys/module/<your-module>/sections/.data

## 其它平台
qemu不是虚拟机，而是一款模拟器。这意味着它可以模拟很多硬件平台。对于多种硬件平台的内核，只需选择对应的`qemu-system-xxx`即可。我恰巧有一块cubieboard的板子，电脑里有它的内核，那么我就这样运行这个内核:

	qemu-system-arm -machine cubieboard -s -S -kernel project/cubieboard/linux-source-3.4.79-sun7i/arch/arm/boot/zImage -append "console=ttyS0" -serial mon:stdio -nographic

可以看到多使用了`-machine`参数，来指定机器类型(是因为SoC的原因？)。

## 出错处理

GDB报错："Remote 'g' packet reply is too long"
处理办法[(参考这个)](http://lists.gnu.org/archive/html/qemu-discuss/2014-10/msg00069.html)：
下载GDB代码，修改remote.c文件注释下面两行:

	6157  //if (buf_len > 2 * rsa->sizeof_g_packet)
	6158  //  error (_("Remote 'g' packet reply is too long: %s"), rs->buf);

然后重新编译安装GDB。
