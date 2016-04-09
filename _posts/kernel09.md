title: 从零构建内核:多任务
date: 2015-02-02 22:45:33
tags: Note
---
在上一个进程的基础上，扩展多任务操作系统。

<!--more-->

## 进程切换过程 ##

第一步，时间中断。从内核代码（最初状态）或者某一进程陷入时间中断的服务例程；

第二步，切换堆栈。从代码一开始运行的堆栈进入PCB里的保存现场用的堆栈。切换的过程使用了预先设定好的TSS结构；

第三步，保护现场。一系列的`push`指令将寄存器的值依次保存进堆栈；

第四步，切换堆栈。从PCB里保护现场的堆栈切换进运行调度函数用的内核运行栈；

第五步，调度程序。运行调度函数，以某种算法选择合适的进程作为`ready_task`；

第六步，切换GDT中的LDT。因为所有进程的LDT共用一个GDT中的描述符，在任务开始前这个描述符需要进行相应修改；

第七步，切换堆栈。将`esp`从内核运行栈中拿出来放到下一个将要运行进程PCB的保存现场栈中；

第八步，恢复现场。一系列的`pop`指令取出各寄存器的值；

第九步，回复执行。retd。

## 实现细节 ##

每个进程需要分别被初始化的内容：

- 进程名
- 进程ID
- LDT
- 各个段寄存器(cs,ds,es,fs,ss,gs)
- eip和esp
- eflags = 0x1202
- ldt selector = 32（进程共用一个GDT中LDT描述符）

所有进程共享的被初始化的内容:

- ready_task指向默认进程表中第一个进程
- tss

时间中断处理例程代码：

        hwint00:
                sub	esp, 4
                pushad
                push	ds
            	push	es
            	push	fs
            	push	gs
            	mov	dx, ss	; ss's DPL is 0 which others want to be
            	mov	ds, dx
            	mov	es, dx
            
            	mov	al, 0x20	; EndOfInterupt
            	out	0x20, al
        
            	inc	dword [k_reenter]
            	cmp	dword [k_reenter], 0
            	jne	.re_enter
            
            ; move to kernel stack
            	mov	esp, (kernel_stack + KERNEL_STACK_SIZE)
            
            	sti
            ; call functions
            	call	scheduler
            
            	cli
            ; leave kernel stack to pcb.regs
            	mov	esp, [ready_task]
            	lldt	[esp + (18*4)]
            	lea	eax, [esp + (18*4)]
            	mov	dword [itss + 4], eax
            	    
        .re_enter:
            	dec	dword [k_reenter]
            	pop	gs
            	pop	fs
            	pop	es
            	pop	ds
            	popad
            	add	esp, 4
            	iretd
    