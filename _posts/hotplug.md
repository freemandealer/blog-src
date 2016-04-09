title: Linux热插拔机制的介绍和应用
date: 2015-05-24 20:47:33
tags: Note
---
热插拔即带电插拔，热插拔功能就是允许用户在不关闭系统，不切断电源的情况下取出和更换损坏的硬盘、电源或板卡等部件。Linux内核支持热插拔的部件有USB设备、PCI设备甚至CPU。Linux的热插拔支持是一个连接底层硬件、内核空间和用户空间程序的机制，且一直在变化，故立文讨论之。

<!--more-->

## 三种热插拔机制
### PCMCIA
1995年，Linux就实现了一种PCMCIA机制[1]——在硬件接入到计算机上时自动加载驱动程序。为了使用PCMCIA，我们需要准备一份配置文件，告诉内核插入什么卡时加载什么驱动。由于这种方式不够动态，David Brownell 提出交了一次patch，这就是`/sbin/hotplug`。

### /sbin/hotplug
#### 为什么hotplug能工作
/sbin/hotplug之所以能自动化，基于以下三个事实[1]：

- 硬件本身会告诉计算机自己是做什么的（就算没有，它也会告诉内核自己的生厂商代码和独一无二的产品代码）

-  驱动程序清楚自己是驱动哪一类设备的

- 内核通过总线底层代码清楚什么时间什么样的设备被接入或移出计算机

#### /sbin/hotplug工作原理
`/sbin/hotplug`的本质是一个脚本。脚本中解析相关参数并调用`modprobe`和`rmmod`完成加载和卸载操作。但是，`/sbin/hotplug`本身是被谁调用的呢？谁给传的参数呢？

设备驱动程序一般不会和这些太底层的kobject/kset家伙打交道，因为更高层次的device,bus和driver把kobject/kset那一层的细节实现都给封装了起来。以device_add为起点，uevent事件被这样产生和传递[2]：

	device_add
	=>	kobject_uevent(&dev->kobj, KOBJ_ADD)
		=>	/* send netlink message */
			... 
			/* 准备参数 */
			argv [0] = uevent_helper;  
			argv [1] = (char *)subsystem;
			argv [2] = NULL;
			...
			/* 内核空间调用用户空间的程序 */
			call_usermodehelper(argv[0], argv,env->envp, UMH_WAIT_EXEC);
			...

下面看看uevent_helper[0]来自何处：

	  char uevent_helper[UEVENT_HELPER_PATH_LEN] = CONFIG_UEVENT_HELPER_PATH;

`CONFIG_UEVENT_HELPER_PATH`其实是空值。可以通过向sysfs接口`/sys/kernel/uevent_helper`写入应用空间程序路径。

### udev
在udev刚开始流行的时候，有一个过渡期。在这个时期，`/sbin/hotplug`和udev同时存在。`/sbin/hotplug`接收到内核的热插拔事件后会执行一系列脚本，其中一个脚本执行了`/sbin/udevsend`，从而让udev的守护进程知悉这一事件[3]。不过现在，有些发行版中`/sbin/`目录已经不存在hotplug和udevsend了。热插拔事件通过netlink由udevd直接接收并全权负责。通过下面这条命令可以查看系统中传递给udevd的热插拔事件：

	udevadm monitor

## 利用热插拔机制实现模块自动加载
### 开机自动加载
将模块.ko文件复制到`/lib/modules/`uname -r`/kernel/modulename.ko` 目录并更新 `/etc/modules`文件即可实现booting阶段自动加载模块。(这是作弊，没有用到热插拔T~T)

### udev实现自动加载
udev的规则文件放在`/lib/udev/rules.d`和`/etc/udev/rules.d`两个目录中，后者的优先权较高：后者目录中的规则文件会覆盖前者中同名文件。下面是一个实际例子：

为udev键盘规则文件60-keyboard.rules开头增加：
	
	ACTION=="add", RUN+="/lib/udev/hello.sh"
	ACTION=="remove", RUN+="/lib/udev/bye.sh"
	
/lib/udev/hello.sh：

	#!/bin/bash
	sudo -H insmod <路径>/hello.ko

/lib/udev/bye.sh：

	#!/bin/bash
	sudo -H rmmod hello

这样插拔USB键盘时就能加载模块。

### MODULE_DEVICE_TABLE实现自动加载
使用udev规则需要修改和创建很多文件。能不能单纯地在模块的源代码级实现自动加载？答案是可以的。


首先使用`MODULE_DEVICE_TABLE`宏注册模块。接着编译模块并将编译后产生的.ko文件拷贝至`/lib/modules/ ｀uname -r｀/`目录下。然后使用`sudo depmod -a`命令将新的模块信息加入`/lib/modules/ ｀uname -r｀/`目录下的modules.alias和modules.dep文件中。

	#define USB_KEYBOARD_VENDOR_ID 0x093a
	#define USB_KEYBOARD_PRODUCT_ID 0x2510
	
	static struct usb_device_id usb_kbd_id_table[] = {
		{ USB_DEVICE(USB_KEYBOARD_VENDOR_ID, USB_KEYBOARD_PRODUCT_ID) },
		{ }
	};
	
	MODULE_DEVICE_TABLE(usb, usb_kbd_id_table);

上面的代码是实现一块键盘连接上计算机后自动加载模块这个功能所需在模块中添加的部分。VENDOR_ID和PRODUCT_ID每个键盘是不一样的，可以把键盘连接在计算机后，使用`lsusb`命令确定键盘的这两个值。如果你需要对每个 USB 设备都响应而不是特定的VENDOR_ID和PRODUCT_ID值, 那么需要创建一个只设置这个 driver_info 成员的入口项[4]:

	static struct usb_device_id usb_ids[] = {
		{.driver_info = 42},
		{} 
	};

如果只想对所有的USB键盘做响应，那么是这样的：

	static struct usb_device_id usb_kbd_id_table[] = {
		{ USB_INTERFACE_INFO(USB_INTERFACE_CLASS_HID,
				USB_INTERFACE_SUBCLASS_BOOT,
				USB_INTERFACE_PROTOCOL_KEYBOARD) },
		{ }
	};
	MODULE_DEVICE_TABLE(usb, usb_kbd_id_table);

看到诀窍了吗？

## 参考文献
[1] [Hot Plug](http://www.linux-mag.com/id/2617/)
[2] [linux下热插拔事件的产生是怎样通知到用户空间](http://blog.csdn.net/bingqingsuimeng/article/details/7924300)
[3] [The Linux Device Model](http://www.linuxjournal.com/node/5604/print)
[4] 《Linux设备驱动程序(第三版)》第13章 USB 驱动
