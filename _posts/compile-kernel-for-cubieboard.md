title: 为cubieboard linux编译内核
date: 2014-04-24 10:57:31
tags: Note
---
本文为我为cubieboard卡片电脑编译内核的记录，目的是作为学习笔记，虽然我尽可能地让文章向教程靠近，希望能够帮助其它想要做这件事的人，但不保证照着我的步骤就一定能成功．

<!--more-->

##获取原料##
	mkdir stage && cd stage
	git clone https://github.com/matson-hall/allwinner-pack-tools.git tools
	git clone https://github.com/matson-hall/linux-sunxi.git linux
	git clone https://github.com/matson-hall/allwinner-buildroot.git buildroot
	git clone https://github.com/matson-hall/uboot-allwinner.git u-boot
	
	(cd tools; git checkout -b stage-3.9 remotes/origin/stage-3.9)
	(cd linux; git checkout -b stage-3.9 remotes/origin/stage-3.9)
	(cd buildroot; git checkout -b stage-3.9 remotes/origin/stage-3.9)
	(cd u-boot; git checkout -b stage-3.9 remotes/origin/stage-3.9)

##设置交叉编译环境##
设置环境变量：

	PATH=/user/local/arm-none-linux-gnueabi-i686-pc-linux-gnu/bin:$PATH
	
检查是否配置成功:

	arm-none-linux-gnueabi-gcc -v
	
注意:$PATH的位置不同结果可能不一样.附修改回环境变量的命令：

	newPATH=$(echo $PATH | sed s#/usr/bin:##)
	PATH=newPATH
	export PATH
	
##编译内核##
	make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabi- cubieboard_defconfig uImage
	
其中`CROSS_COMPILE`选项写配置好环境变量的交叉编译器的前缀． `cubieboard_defconfig`的意义是在`ARCH=arm`配置后，系统会自动进入`ARCH/arm/configs`里去找`cubieboard_defconfig`作为配置文件进行编译．如果不显式声明`ARCH`,默认为宿主架构.
##结果##
	Kernel: arch/arm/boot/zImage is ready
	UIMAGE: arch/arm/boot/uImage
	"mkimage" command not found - U-Boot images will not be built
	make[2]: *** [arch/arm/boot/uImage] 错误 1
	make[1]: *** [uImage] 错误 2
	make: *** [uImage] 错误2
##未来工作##
这次为cubieboard编译内核主要目的是练习，为后面给Android手机（非虚拟机）编译安装内核做准备．内核编译完成后还需要进行安装．详细过程将在不久后的给真机编译及安装内核的笔记中描述．这里做一个预告．
