title: 从零构建内核:中断
date: 2015-01-14 10:22:12
tags: Note
---

构造中断主要分

<!--more-->

## 初始化中断描述符表寄存器(idtr) ##
使用如下结构去填充idtr:

    16bit Limit
    32bit base
    
在汇编中使用指令`lidt`加载（首地址做操作数）。注意这里用C语言描述该结构不能用如下方式:

    struct idtr {
        u16 limit;
        u32 base;
    };

因为这样会因为C语言内存对齐的特性造成在两个域之前留下一个16bit的空白。应该使用：

    struct idtr {
        u16 limit;
        u16 baselow;
        u16 basehigh;
    };
    
再用与和移位等位操作进行初始化。另外:

    struct idtr {
        u16 limit;
        u32 base;
    } __attribute__ ((packed));

中的`__attribute__ ((packed))`可以让编译器压缩结构体到最小，从而避免对齐的发生。

## 设置8259A ##

    out_byte(0x20,  0x11);                                                    
    out_byte(0xa0,  0x11);                                                    
    out_byte(0x21,  0x20);  硬件中断主从0x20开始
    out_byte(0xa1,  0x28);  硬件中断主从0x28开始                                                  
    out_byte(0x21,  0x4);                                                     
    out_byte(0xa1,  0x2);                                                     
    out_byte(0x21,  0x1);                                                     
    out_byte(0xa1,  0x1);                                                     
    out_byte(0x21,  0xfd);   设置屏蔽位，这里只打开了键盘中断0x2                                                 
    out_byte(0xa1,  0xff);  

## 建立IDT ##

IDT一个项用C语言描述长这样：

    struct idt_entry {                                                                
        u16 offset_low;                                                           
        u16 selector;                                                             
        u8 dcount;                                                                
        u8 attr;                                                                  
        u16 offset_high;                                                          
    };
    
在我的内核里，IDT共有`IDT_SIZE=256`个项目。存放在叫idt的全局数组中。

## 初始化门 ##
(待补充）
## 特权级别转换和TSS初始化 ##
(待补充)

## 调试中断 ##
调试中断设置主要用到的bochs调试命令：

- sreg: 参看段寄存器（idtr被归到这里了）
- info idt: 参看idt信息