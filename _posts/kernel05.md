title: 从零构建内核:局限
date: 2015-01-13 18:30:22
tags: Note
---
内核设计中有些部分是野路子,目的只有一个:简(tou)单(lan).不优雅的代码给目前的内核带来了一些局限(不过这些局限对目前的内核却没什么影响:P).写此博文,提醒自己,以及便于以后扩展.

<!--more-->

- 内核编译提取后大小不能超过1K,超过的话需要修改`loader.asm`多读扇区.但是:
- 内核+全局变量的空间最大不应超过607KB
- 目前使用的栈是`0x7c00`向下的长度为30464B(约30KB)的空间
- `0x7e00`及其向后的若干字节存放着BIOS信息
- c语言开发内核时用到的全局变量不能被自动初始化
- 页目录表页表等存在`0x100000`开始的若干内存空间中
- `0x7dfc`开始的若干空间用来作为debug时的输出空间
![当前内核物理内存布局](/img/kernel-mem-layout.jpg)
