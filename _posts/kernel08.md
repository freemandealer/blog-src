title: 从零构建内核:第一个进程
date: 2015-01-28 17:55:31
tags: Note
---
“内核”上成功运行了第一个进程。本文只涉及最基础的进程概念，主要探讨进程的要素和从高特权级跳转低特权级的方法。可以预告的是后面将紧接着进行多进程实验。

<!--more-->

## 进程控制块(PCB) ##

扯淡一分钟：本科保研资格面试，抽到这个题目：PCB是什么？包含什么内容？当时我就意识到这个题目最大的难点就在于，在关于PCB我知道的比在座所有考官加起来还要多的情况下如何保持谦虚和礼貌 :P

扯淡完毕。出于拿来主义,我现在的PCB(TCB)长这样:

    struct regs {
        	u32 gs;
        	u32 fs;
        	u32 es;
        	u32 ds;
        	u32 edi;
        	u32 esi;
        	u32 ebp;
        	u32 kernel_esp;
        	u32 ebx;
        	u32 edx;
        	u32 ecx;
        	u32 eax;
        	u32 retaddr;
        	u32 eip;
        	u32 cs;
        	u32 eflags;
        	u32 esp;
        	u32 ss;
    };
    
    struct tcb {
        	struct regs regs;
        	selector_t ldt_sel;
        	struct descriptor ldt[LDT_SIZE];
        	u32 pid;
        	char p_name[16];
        	u16 stack[TASK_STACK_SIZE];
    };
    
其中`struct regs`用来保护现场。

## LDT ##

新的进程需要自己的地址空间。之前的GDT中的描述的都是DPL0的段。我需要构建新的DPL1的段。不需要大动干戈地再搞一个GDT，一个局部LDT因地制宜。而且可以放进TCB被进程带着走。逻辑上和方便成度上都是合理的。

之前GDT是这样的（汇编）：

    LABEL_GDT:		boot_descriptor	0,	0,	0
    LABEL_DESC_FLAT_C:	boot_descriptor	0,	0fffffh,DA_32|DA_CR|DA_LIMIT_4K
    LABEL_DESC_FLAT_RW:	boot_descriptor	0,	0fffffh,DA_32|DA_DRW|DA_LIMIT_4K
    LABEL_DESC_VIDEO:	boot_descriptor	0B8000h,0ffffh,	DA_DRW|DA_DPL3
    LABEL_DESC_LDT:		boot_descriptor 0,	0,	0
    LABEL_DESC_TSS:		boot_descriptor 0,	0,	0

其中GDT[0]是留白。GDT[3]是视频段，DPL为３，够低了。GDT[4]和GDT[5]是LDT和TSS的描述符，属于系统段/门描述符(S位置零)。GDT[1]和GDT[2]是代码段和数据段，是新进程需要的。
我只改代码段和数据段的DPL,应该是这样的(C语言)：

    init_descriptor(&(idle_task->ldt[0]), 0, 0x0fffff, DA_32 | DA_LIMIT_4K | DA_C | DA_DPL1);
    init_descriptor(&(idle_task->ldt[1]), 0, 0x0fffff, DA_32 | DA_LIMIT_4K | DA_DRW| DA_DPL1);

最后，修改GDT[4](LABEL_DESC_LDT)，登记LDT：

    	init_descriptor(&gdt[4], (u32)(idle_task->ldt), sizeof(struct descriptor) * LDT_SIZE - 1, DA_LDT);

## 选择子 ##
    
这里的选择子有：

TCB里的ldt_sel：在GDT中选择LDT描述符。

赋值给tcp中regs中段寄存器的选择子：在LDT中选择代码段或数据段。

    	cs_sel = (0 & SA_RPL_MASK & SA_TI_MASK) | SA_TIL | SA_RPL1;
	ds_sel = (8 & SA_RPL_MASK & SA_TI_MASK) | SA_TIL | SA_RPL1;
	es_sel = (8 & SA_RPL_MASK & SA_TI_MASK) | SA_TIL | SA_RPL1;
	fs_sel = (8 & SA_RPL_MASK & SA_TI_MASK) | SA_TIL | SA_RPL1;
	ss_sel = (8 & SA_RPL_MASK & SA_TI_MASK) | SA_TIL | SA_RPL1;
	
这类选择子TI位置位，RPL1。告诉系统是LDT中的段，RPL为任务级。
	
视频选择子：在GDT中选择视频段，但RPL=1。

	gs_sel = (24 & SA_RPL_MASK) | SA_RPL1;

TSS选择子：在GDT中选择TSS段。

另外关于选择子，前16-3位是GDT项相对于GDT头的偏移量，因为GDT项大小是8Byte，所以它是8Byte对齐，选择子后三位始终是零。充分利用空间，INTEL大叔在后三位中保存IT（是不是LDT段）和RPL。

![选择子](/img/kernel_selector.png)


## TSS和堆栈切换 ##

因为之前的代码都是运行在高特权级(0级)下，而进程需要运行在较低的特权级（实验用了1级）。所以要进行跳转。跳转的技巧就是retf指令——假装一次中断的完成，调“回”Ring1（原来我们从一开始都只是一个中断，这一定是个大阴谋。。）

跳转牵扯到堆栈的变化。目前我们是从内层到外层（ring0->ring1），ss和esp应该准备好在堆栈中，retf以后自动送寄存器。

    ...
    	idle_task->regs.ss = ss_sel;
	idle_task->regs.gs = gs_sel;
	idle_task->regs.eip = (u32)idle_task_main;
	idle_task->regs.esp = (u32) idle_task->stack + TASK_STACK_SIZE;
	...
	
我们看到eip也是被准备好了。然后：
    
    ; void start_idle()
    global start_idle
    start_idle:
        	mov	esp, [idle_task]
        	lldt	[esp + (18*4)]	; ldt_sel offset in struct tcb
        	lea	eax, [esp + (18*4)]	; stack offset in struct tcb
        	mov	dword [itss + 4], eax	; esp0 offset in tss, set it to be
        					; the bottom of regs in struct tcp
        					; prepare for next 'real' interupt
        	pop	gs
        	pop	fs
        	pop	es
        	pop	ds
        	popad
        
        	add	esp, 4 ;skip retaddr
        	iretd

其中有意思的是，我不仅完成了内层到外层相关的堆栈转换，还为下一次真正的中断时外层到内层的转换埋下伏笔：将`TSS`中的`esp0`初始化为`TCB`中用来保存现场的结构`struct regs`。这样，下次的中断过程中，堆栈就是这样变换了: 进程栈－>TCB保存现场的结构(regs)->内核栈。

## Debug感悟 ##

这个实验思路还算清晰，但设计内容比往常多。自己也是一口气写了下来，结过debug了一整天。有些诡异的错误不知怎么地就好了。

其中发现，因为异常和中断已经初始化，程序出错不会再“跑飞”了——一切尽在掌握。异常处理时可以在handler中通过tmp_dbg获得异常向量和异常代码，结合Intel开发手册，帮助排错。

在debug过程中，把之前保护模式中一些模棱两可的概念和操作搞细致了，也算收获颇多，不然也没这么多写成这篇文章。



现在“内核”真的是内核了！一开心，画了幅画,Happy coding~

![歌剧厨妈](/img/singer.jpg)







