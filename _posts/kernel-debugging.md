
title: Linux环境下通过Vmware调试内核及模块
date: 2015-03-18 18:57:34
tags: Note
---
![](/img/NCIC.jpg)
*本文结构和内容都很平坦（就是说很长，大家耐心点！）目标读者：在Linux环境下调试VMware中kernel及module的技术人员*

<!--more-->

来实习一周了，了解了一些理论准备搭建个开发内核驱动环境试试。刚改好一个网卡驱动的框架，加载进内核一会儿就崩了。我问志国师兄有没有特殊的内核调试技巧。
    “printk啊！”
师兄萌萌哒！不过，printk大法感觉要调一万年！调到电脑要爆炸！我去网上搜了一艘，发现print大法确实是一种专业的调试方法（Log调试技术），不过主要应用于开发后期性能瓶颈的排查。对于我这样的内核崩溃，系统直接冻住了，根本没有办法看Log。机智的我赶紧`ctrl+alt+F2`切进TTY：在这里，内核会在崩溃前倾尽全力把肚子里的东西吐出来(Core Dump或者单纯的dmesg)让我们去分析它的死因。不过，吐得太猛，屏幕太小了，还不能翻页＝＝

OK，是时候展现真正的调试技术了——使用专业的调试器/调试补丁。LDD3中第四章一下就讲了仨：kdb，kgdb和kgdb(名称一样)。调试得有两台逻辑上机器，一台host，一台target。既然实验室没有配机器，我也只能求助虚拟机/模拟器了。虚拟机/模拟器又有那么多种：VMware, VirtualBox, Xen, KVM, Qemu（GoldFish）, Bochs。到底用哪个？手头上的最好用！

## 我的环境是这样的 ##

HOST(运行GDB): 物理机器，Ubuntu14.04x86_64
TARGET（运行被调试的内核）: 虚拟机(in VMware 6.0)，CentOSx86_64，KernelVer=2.6.32

注意架构HOST和TARGET要一致（这里都是x86_64），否则HOST的gdb无法识别TARGET的内核二进制文件的格式，强行调试的话没准得整个交叉编译工具链里的gdb。

## 构建TARGET ##
在VMware里安装TARGET操作系统。这个系统就是将来被各种解剖的系统。下面需要替换内核，挺麻烦的。但是替换内核的理由完全盖过了复杂性：为了开打调试开关，为了增加调试信息，为了解除内核结构体保护等等。理由很多，但是我们只需要一条：求个踏实。我们的目的是KernelHacking，连个自己的内核都没有，遇到麻烦怎么能说“一切尽在掌握中”呢？我们着手做吧！

获得内核、编译内核、配置内核和安装内核的详细教程网上颇多，考验大家的"google-fu"。这里只提醒大家一些需要注意的地方。

### 这些开关要打开 ###
下面是配置内核阶段需要打开状态的开关，可以通过在内核源代码目录中执行`make menuconfig`或`vi .config`，再加上一些搜索命令确认一下打开情况。

    ...
    CONFIG_MAGIC_SYSRQ=y
    ...
    CONFIG_DEBUG_INFO=y
    ...
    CONFIG_KGDB=y
    CONFIG_KGDB_SERIAL_CONSOLE=y
    ...

### 这些要关闭 ###
内核中有一些磨人的小妖精，平常好端端的，但对于我们KernelHacking就不是很友好了。在调试环境搭建时，请确保R(ead)O(nly)DATA是关闭的，不然内核数据结构受保护，到时候只能眼睁睁地看着内核在跑，没法打断点，访问值。

    # CONFIG_DEBUG_RODATA is not set

### 编译安装 ###

    make -j4 && make -j4 modules && sudo make modules_install && sudo make install && make -B vmlinux
    
这一大串指令将编译内核和模块，然后安装内核和模块。最后一个vmlinux需要留心，它是我们祭祀给gdb的ELF文件，gdb就是从这里获得内核符号的。明眼的同学突然回忆起来："我看到`/boot/`目录底下有个vmlinux-XXX文件来着..."不能用那个！`/boot/`下面的二进制镜像是精简压缩过的内核镜像，我们这里的`vmlinux`大小是前者的十来倍，包含了更多信息！

## 连接HOST和TARGET ##
我使用了VMware提供的虚拟串口和主机通信，完成“远程“调试。

首先需要确保虚拟机处于关闭状态，再在硬件配置里为虚拟机添加`Serial Port`。

![Add Serial Port](/img/add-serial-port.png)

因为调试需要交互，使用第三种｀Use Socket(named pipe)｀连接方式。文件路径和名称可以自定义。核心技术是｀from server to application｀要选对。

操作完成后，启动虚拟机系统，`/tmp/`中出现了`dbg_pipe`文件。下面我们测试联通性。

    # 在HOST上 #
    socat /tmp/dbg_pipe TCP4-LISTEN:9001　#将文件映射到一个端口
    telnet 127.0.0.1 9001


    # 在TARGET上 #
    sudo chmod 222 /dev/ttyS1
    echo 'hello' > /dev/ttyS1

这时在HOST的telnet回话中会显示`hello`。这里的`ttyS*`到底是几每个机器不一样。在VMware创建虚拟机的默认过程，串口打印机占用了`ttyS0`。所以在默认情况下，对于新装的虚拟机，第一个手动添加串口设备文件对应`ttyS1`。

反过来，从HOST向TARGET传送数据：

    # 在TARGET上 #
    sudo cat /dev/ttyS1
    
此时在HOST上的telnet回话中敲入一些字符，这些字符将在TARGET的终端里显示。

## 调试! ##

内核调试目的有多种，大体上分两类：一是学习内核运行流程，再者是编写驱动时debug。面对这两类情况，调试内容也得分为：调试内核本身和调试模块两种情况。

### 调试内核自身 ###
*注意：执行上述操作前确认`socat /tmp/dbg_pipe TCP4-LISTEN:9001`在执行。*

首先，修改内核参数`/boot/grub/`目录中的`grub.cfg`或是`menu.lst`（需要root权限）。在kernel或linux指令后追加参数：`kgdboc=ttyS1,115200`。有时候我们需要调试内核启动过程，需要内核等着调试器接管后再启动，这种情况下可以加`kgdbwait`参数。这样系统启动时，如果没有调试器接入并发送`c(ontinue)`命令，就停在那儿等。

进入系统后，我们需要断下内核。

    # TARGET上执行 #
    sudo chmod 222 /proc/sysrq-trigger
    echo g > /proc/sysrq-trigger

这样，内核就会暂停运行，等待调试器接管、给出指令。

    # HOST上执行 #
    gdb ./vmlinux
    ...
    (gdb) set serial baud 115200
    (gdb) target remote localhost:9001
    
>TIP: 合理使用`gdb -x <script>` 可以利用脚本省去每次debug前输入这么多gdb命令的麻烦

*再一次注意：执行上述操作前确认`socat /tmp/dbg_pipe TCP4-LISTEN:9001`在执行。*

不过存在一疑问：上面的这个`vmlinux`是个啥？这就是上面编译安装部分讲到的那个`vmlinux`，我们需要把它从TARGET中拷贝到HOST上。

### 调试模块 ###

有时候我们不是真的想调内核。。。我只是个写驱动的，只想调那个insmod加载进去的驱动模块。好，现在我们就来调模块。

我们知道Linux是宏内核，内核和所有模块都运行在同一地址空间，这为我们调试内核提供了便利——大家都在一起！但是，我们还是不能直接用上面的方法调试内核模块。如果你的模块中提供了一个`foo`函数，你直接`break foo`是没法在这个函数的入口处打断点的。系统会提示你找不到符号，就算去掉`static`修饰、增加`SYMBOL_EXPORT(foo)`也不行——原因不在这儿！仔细想想GDB是从vmlinux文件中获得符号，而模块的符号并不包含在vmlinux中（vmlinux建立时模块甚至可能都不存在），难怪！模块的符号都存在ko文件里。知道这些，我们导入这个文件行了。

    # 在GDB中执行 #
    (gdb) add-symbol-file <your-module.ko> <address>

其中`<your-module.ko>`就是编译后模块文件的名称，`<address>`是什么呢？是模块代码段的加载地址，就是一个偏移量。GDB就是通过这个偏移量算出每个符号在运行内核中的地址。模块加载地址这样查看：

    sudo cat /sys/module/<your-module>/sections/.text

### 试一试 ###

让我们断下内核，接入GDB，敲几个命令试一试：

    # HOST上输入命令 #
    (gdb) break sys_mount
    (gdb) continue
    
    # TARGET上输入命令 #
    sudo mount /dev/sda1 /mnt/

输入完成后，被调试的系统被中断，接着输入'n'单步执行。

    (gdb) n

结果没发现源代码？而是提示一个路径，说找不到源代码文件？

诶！可怜的GDB不知道自己调试的是远程主机上的内核程序，还在拼命在本机上找源文件呢？不知道有没有什么命令可以告诉GDB这个坏消息。。我采取了将计就计的办法，把TARGET上的源文件拷贝到HOST上，布局成TARGET机器上的样子，让GDB去找。没想到GDB信以为真，认真地打印出一行一行的源代码。

## 降低优化程度 ##

至此，调试环境基本搭建完成！但是由于内核编译时优化过猛，破坏了二进制代码和源代码对应关系，调试时有些忧郁。下面需要找出降低优化程度的办法。

    # Makefile里头加这个 #
    ifeq ($(DEBUG),y)
        DEBFLAGS = -O -g -DSBULL_DEBUG
    else
        DEBFLAGS = -O2
    endif
    EXTRA_CFLAGS += $(DEBFLAGS)

**HAPPY HACKING!!**
