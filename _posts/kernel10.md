title: 从零构建内核:中断处理框架
date: 2015-02-17 21:34:51
tags: Note
---
中断处理程序，也叫Interupt Handler或中断服务例程(ISR)，是中断发生后运行的一系列代码，一般用作对中断的响应。在我的内核中，构建了一个简单的框架，方便新处理函数的加入。

<!--more-->

## 加入新ISR ##

- 编写一个处理程序，参数为一个整型IRQ号
- 调用`add_isr()`，参数依次为irq号和处理函数的地址

在内核中，`add_isr`可以在任何地方调用，但初始化内核时集中在interupt.c文件中的setup_idt()中以便于管理。

## 框架原理 ##

### isr_tbl ###
`isr_tbl[ISR_NUM]`是一个`int_handler`也就是函数指针类型的数组，存放着ISR，IRQ号顺序对应。内核最多支持`ISR_NUM`个中断类型。事实上`add_isr`函数就是在操作这个数组。

### 汇编头 ###
`isr_tbl`中的函数只是中断处理的一部分，现场保护和堆栈切换都在interupt_entry.asm文件中用汇编实现。以时间中断(0号中断)为例：

            sub     esp, 4
            pushad
            push    ds
            push    es
            push    fs
            push    gs
            mov     dx, ss          ; ss's DPL is 0 which others want to be
            mov     ds, dx
            mov     es, dx
    
            in      al, 0x21        ; block same interupt source
            or      al, (1<<**0**)
            out     0x21, al
    
            mov     al, 0x20        ; EndOfInterupt
            out     0x20, al
    
            inc     dword [k_reenter]
            cmp     dword [k_reenter], 0
            jne     .re_enter**0**
    
            mov     esp, (kernel_stack + KERNEL_STACK_SIZE)
    
            sti
            push    **0**
            call    (isr_tbl + 4 * **0**)
            add     esp, 4
            cli
            
            mov     esp, [ready_task]
            lldt    [esp + (18*4)]
            lea     eax, [esp + (18*4)]
            mov     dword [itss + 4], eax
    .re_enter**0**:
            in      al, 0x21        ; restore the same interupt
            and     al, ~(1<<**0**)
            out     0x21, al
    
            dec     dword [k_reenter]
            pop     gs
            pop     fs
            pop     es
            pop     ds
            popad
            add     esp, 4
            iretd
