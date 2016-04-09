title: 从零构建内核:BootLoader
date: 2015-01-02 19:18:24
tags: Note
---

一个简单的Loader代码

<!--more-->

    [SECTION .loader]
        	org	07c00h
        	mov	ax, cs
        	mov	ds, ax
        
        	xor	ax, ax
        	xor	dl, dl
        	int	13
        
        	mov	ax, kernel_base
        	mov	es, ax
        	mov	ah, 02h
        	mov	al, 01h	
        	mov	ch, 00h
        	mov	cl, 02h
        	mov	dh, 00h
        	mov	dl, 00h
        	mov	bx, 0
        	int	13h
        	jmp	kernel_base_phy
        
    times	510-($-$$)	db	0
        dw	0xaa55
        
这段代码编译后保存在磁盘的引导扇区。x86架构的计算机在开机阶段BIOS将会将这个扇区的代码复制到`0x7c00`这个地址开始的内存区域，并跳转到这个地址开始执行。BIOS只会复制这个扇区的内容（512字节的内容）无论代码编译后有多长。这个512字节能实现的功能有限，我们只用这段代码来加载更多的代码。

具体说，该代码调用BIOS功能，初始化磁盘驱动器。然后再次调用BIOS功能，将引导扇区后面几个扇区的代码复制到内存指定地址(kernel_base_phy = kernel_base<<4)。并将控制权交给这些代码（跳转过去）。

如何构造后面扇区的代码？

可以选择在loader后面接着编写代码。因为loader正好是512字节。后面的代码将会被保存在第二个以及后面的扇区。但是代码在编译时，我们用`org`指令告诉编译器这段代码将会装载到`0x7c00`的地方执行。然而我们却把后面的指令转移到别处去执行了。这样带来的问题是汇编中的标号（地址）对应不上实际地址。可以通过相对的偏移量解决这个问题，但是过程复杂并容易出错，代码可读性也不好。

那为什么不在后面再写一个`org`指令指定新的基准呢？汇编语言并不允许一个源文件中存在两个`org`。虽然行不通，但是给了我们一个启发：写两个源文件————一个文件编译好就是loader放引导区，另一个编译后在磁盘上紧接在引导扇区后面。第二个文件开头：

    org	kernel_base_phy
    ......

我们需要用`dd`命令将这个文件编译后的机器码拼接成我们需要的形式。

    dd if=loader.asm of=a.img conv=notrunc
    dd if=other.asm of=a.img conv=notrunc bs=512 seek=1
    
后一句跳过a.img前512字节，写入other.asm。