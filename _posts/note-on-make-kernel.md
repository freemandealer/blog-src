title: 内核模块编译安装杂记
date: 2015-08-08 22:18:12
tags: Note
---
> 真男(tu)人(hao)敢于用实体机做内核实验，而善良的人用虚拟机，善良的真男人给虚拟机徒手装内核！ ————　我刚说的

<!--more-->


## 徒手装内核! ##
虚拟机编译好慢好慢的，实验过程不爽快！所以之前在实体机上编译模块，传到虚拟机上测试。现在延生出了一个想法，就是在实体机上编译整个内核，然后到虚拟机上安装——也就是模拟了`make install`和 `make modules_install`的过程。

查看`/boot/grub.cfg`文件我们得知Grub启动系统需要两个文件：vmlinuz-x.y.z-m和initrd.img-x.y.z-m。但是人家也喜欢把System.map-x.y.z-m放在`/boot`中，不晓得有什么用，我们也放一个，反正不麻烦。

获得vmlinuz的方法很简单，make完后`arch/x86_64/boot/`里的`bzImage`其实就是它，改个名字vmlinuz-x.y.z-m就好，其中x.y.z是版本号，m是魔法串。

获得initrd.img文件过程有一些坎坷。它是由这个命令产生的：

	sudo mkinitramfs -o initrd.img-x.y.z-m x.y.z-m

在`/boot`目录执行这个命令会报错，提示你需要准备`/lib/modules/x.y.z-m/`目录。你需要在这个目录中提供modules.order文件和modules.builtin文件。你可以在编译源代码的目录中找到它们并拷贝过去。你还需要准备`/lib/modules/x.y.z-m/kernel`，让里面装着对应版本的内核模块文件，否则重启后连根文件系统都挂载不了，只能在initramdisk的busybox环境里飘荡(好像WindowsPE :( )。为了这些内核模块，我把整个源代码目录备份一遍，然后把不是*.ko的文件都删掉减小空间，然后拷贝到`/lib/modules/x.y.z-m/kernel`。

最后修改`grub.cfg`文件为我们的内核增加一个启动项。

## 应万变的命令 ##
如果要为一个已有内核编译模块，那过程将会很让人懊恼，这个老教程可能会给你一些提示。你需要编译一个模块，那么你最好安装自己的内核。然后你可以：1)脱离树独立编译 2)make menuconfig后整体编译(因为你已经编译过内核了，所以make的差量构建性质会使得这个过程快很多)但更好的办法是使用这条以不变应万变的命令：

	make -C <KERN-DIR> M=<MODULE-DIR> [CONFIG_XXX=m]

如果想单独编译一个目录

	make /xxx/yyy/zzz

但上述命令只能获得.o中间文件，要想得到.ko模块文件，需要执行：

	make M=/xxx/yyy/zzz

