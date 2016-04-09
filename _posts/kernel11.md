title: 从零构建内核:系统调用
date: 2015-02-28 10:08:41
tags: Note
---
内核工作在最高特权级上。由于x86保护机制的存在，低特权级的用户态进程不能使用一些特权指令，或者访问内核数据，以保证系统的稳定。但有时候，我们需要提供用户态进程访问内核资源的途径。但这个途径必须是事先安排好的指定路线——去内核这个野生动物保护区，出于游客安全和动物保护的原则，游客不应该随意行动。系统调用就是一辆辆旅游观光车，以内核的身份代替用户进程访问内核数据或者执行内核代码。

<!--more-->

## 系统调用过程 ##
在本内核中，系统调用实现涉及到的源文件和过程如图所示：

![系统调用过程](/img/system_call_workflow.jpg)

使用系统调用和使用正常函数没有任何区别，直接调用sys_call_entry.asm中的函数即可。如果带有参数，则正常传参。

sys_call_entry.asm中的被调函数主要任务是参数处理、存储系统调用号到eax寄存器、执行`int 0x80`指令。因为所有的系统调用都是`idt[0x80]`对应的处理函数处理的，需要用系统调用号来指定具体的系统调用。


`idt[0x80]`对应的处理函数，这里是位与interupt_entry.asm中的`sys_call`。它保护现场，然后用eax寄存器中的系统调用号索引`sys_call_tbl`表。

sys_call_tbl定义在sys_call.c中，它的元素类型为`sys_call_handler_t`，其实是一个`void *`，这样可以保证在任意返回值类型，任意参数个数和类型的情况下能被正确调用。

图中以`get_ticks`为例，系统调用号是0，所以表中的第一项`sys_get_ticks`被执行。

## 返回值 ##
`sys_get_ticks`运行结束后，返回值存放在eax中，而eax将会被`int 0x80`处理函数(sys_call)末尾用来恢复现场的`popad`修改，导致返回值丢失。有的系统调用是需要返回值的，所以得先把PCB保护现场堆栈中的eax的值用当前存有返回值的eax替换掉。这样系统调用就能正常返回值了，好像所有工作都是get_ticks自己做的一样。

## 添加系统调用 ##
了解了本内核的系统调用原理和细节，本节说明如何添加一个系统调用。因为没有形成一个系统话的系统调用框架，需要手动修改多个文件：

- 找一个空闲系统调用号，参考sys_call.c和sys_call_entry.c开始部分
- 在sys_call_entry.asm中实现一个 `mov eax, 系统调用号; int 0x80`,如果有参数，可以使用寄存器传参
- 实现一个`sys_`函数，用来处理系统调用
- 将上述`sys_`函数指针加入`sys_call_tbl[系统调用号]`的位置
- 调用sys_call_entry.asm中的函数以使用新建的系统调用。