title: 用QEMU来调试内核 -- 亲身体验篇
date: 2015-10-04 13:37:39
tags: Note
---
本文记录了用QEMU构建最小Linux系统并进行调试的过程。

<!--more-->

##愿景

我在[用QEMU来调试内核 -- 亲身体验篇](http://freemandealer.github.io/2015/08/27/debug-kernel-with-qemu/)中大致记录了邮件列表上和网上搜索到的内核调试方法，并没有完全进行验证。今天亲自实践了一番，发现：

- QEMU调试果然爽快
- 调试环境搭建过程中有很多细节需要注意

先来看看有了QEMU内核调试环境后，我调试内核的大致步骤。

- 修改内核，make编译（不用完整执行，生成bzImage就可以中断make）。执行

	 	qemu-system-x86_64 -kernel arch/x86_64/boot/bzImage -initrd initramfs-busybox-x86.cpio.gz

- 运行GDB(这个GDB是要自己编译的，见[上篇文章的末尾](http://freemandealer.github.io/2015/08/27/debug-kernel-with-qemu/))。

		gdb vmlinux
		> target remote localhost:1234

- 进入弹出的QEMU窗口，`CTRL+ALT+2`进入QEMU控制台，输入`gdbserver`。

- 调试开始！

再也不用像使用VMware那样等一万年去`make modules&&make install &&make modules_install`以及费尽心思去调教grub了。节省了大量时间，同时资源占用也少太多。一切都是整整齐齐的模样。

本文采取倒叙的描述方式，上面就是我们的目标啦。怎么才能实现呢？参考LFS，我们进行下面的步骤。

##为什么要构建自己的运行系统

插一段话，解释什么是LFS。LFS是[Linux From Scratch](http://www.linuxfromscratch.org/lfs/view/stable/index.html)缩写，意思是利用网络上的开源代码，从头**构建**Linux**发行版**。“**构建**”这里指编译和安装，不是指设计程序和敲代码的过程。构建的目标Linux**发行版**，不是让你从头写Linux内核本身。Linux发行版有很多外延，比如CentOS、Ubuntu、Arch等等。我自个儿就琢磨着呗，我们做内核开发，不能老用别人的发行版：一来我们要分清什么是Linux的共性特征，什么是一些发行版加进去的个性特征；二来从头构建Linux发行版能加深我们对这个系统的理解。更重要的是，毕竟我们研究内核其实就是在溯源，我不希望在探求原理路途上有什么迷雾遮住自己的眼，而是喜欢“一切都在掌握之中”的良好感觉。

## 环境搭建

开始的内核编译我已经不想再说了。使用默认配置文件即可：

	make x86_64_defconfig

看最上面运行QEMU的那条命令，其实我们缺的就是initramfs-busybox-x86.cpio.gz文件——一个Initial RamDisk——开机时需要挂载的rootfs和一些二进制工具以及最重要的init。

有两种方式获得这个Initial RamDisk。一种方式是在内核编译前生成，另一种通过内核编译选项在内核编译后生成。

### 方法一：编译前生成

先用下面的命令下载并解压缩BusyBox:

	curl http://busybox.net/downloads/busybox-1.23.2.tar.bz2 | tar xjf -

为BusyBox创建工作目录:

	mkdir -pv obj/busybox-x86
	make O=obj/busybox-x86 defconfig

使用menuconfig配置BusyBox:

	make O=obj/busybox-x86 menuconfig

选择静态链接:

	-> Busybox Settings
		-> Build Options
		[ ] Build BusyBox as a static binary (no shared libs)

编译、安装BusyBox:

	cd obj/busybox-x86
	make -j2
	make install

拷贝安装目录中的工具到initramfs目录中，这个文件夹就是日后的initramfs:

	mkdir -p initramfs/x86-busybox
	cd initramfs/x86-busybox
	mkdir -pv {bin,sbin,etc,proc,sys,usr/{bin,sbin}}
	cp -av obj/busybox-x86/_install/* 

initramfs目录中没有init脚本，这可不行，我们的内核起来以后运行什么程序呢？我们手动创建一个：

	#!/bin/sh
	
	/bin/mount -t proc none /proc
	/bin/mount -t sysfs sysfs /sys
	echo 'Enjoy your new system!'
	/bin/sh
	
生成initramfs:

	find . -print0 | cpio --null -ov --format=newc | gzip -9 > initramfs-busybox-x86.cpio.gz

得到initramfs了，任务结束了!看明白了吗？

同时这里给出生成initramfs的逆操作－－拆分initramfs:

	cpio -i -d -H newc -F initramfs_data.cpio --no-absolute-filenames

###方法二：编译后生成

通过内核编译前配置`CONFIG_INITRAMFS_SOURCE`选项到一个存在的gzipped的initramfs、或是一个准initramfs目录、或是如下格式指定initramfs结构的txt文件:

	dir /dev 755 0 0
	nod /dev/console 644 0 0 c 5 1
	nod /dev/loop0 644 0 0 b 7 0
	dir /bin 755 1000 1000
	slink /bin/sh busybox 777 0 0
	file /bin/busybox initramfs/busybox 755 0 0
	dir /proc 755 0 0
	dir /sys 755 0 0
	dir /mnt 755 0 0
	file /init initramfs/init.sh 755 0 0


##背后的原理

## 参考

如果你想弄清楚内核的初始化过程，这里有一份阅读列表：

[How to Build a Custom Linux Kernel for Qemu](http://mgalgs.github.io/2012/03/23/how-to-build-a-custom-linux-kernel-for-qemu.html)

[Linux From Scratch](http://www.linuxfromscratch.org/lfs/view/stable/)

[Kernel Doc：Ramfs, Rootfs and Initramfs](https://www.kernel.org/doc/Documentation/filesystems/ramfs-rootfs-initramfs.txt)

[Kernel Doc: Early Userspace Support](https://www.kernel.org/doc/Documentation/early-userspace/README)