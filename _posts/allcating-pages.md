title: Linux如何分配页
date: 2015-08-12 20:15:29
tags: Note
---
是的，在这里记录`alloc_pages`相关的内容！先来看看我们怎么使用它的。`alloc_pages`函数族为我们分配2^n个页（准确讲是页框，即4K大小的物理内存），然后返回页描述符（描述页框状态的数据结构，`struct page`）。可是我们申请内存，是为了使用内存。使用内存时，寻址用的是线性地址。`alloc_pages`返回的描述页框数据结构实例的地址，并没有什么直接用途。我们要把页描述符描述的物理页框的物理地址转换成线性地址，才能被内核使用。为了完成这个“映射”任务，我们调用`kmap`函数族，最终获得了4K线性内存空间的首地址。

<!--more-->

简单说，上述过程可以描述为：

- 获得物理页返回页描述符地址(alloc_pages)
- 修改页表增加映射(set_pte)
- 返回页的线性地址(kmap)

我撒谎了，第一段描述中可没有体现出第二步。且听我慢慢给大家解释：物理内存被分为三类区域：DMA专用的区域、高端内存使用的区域和剩下的普通区域。所有的物理内存在系统初始化时，被切成片片，每一片描述为一个`struct page`，用一个叫`mem_map`的数组把所有的`struct page`装起来。初始化时建立的页表映射了前896MB内存，也就是前229376个页框。这个映射非常规范:

	线性地址 = 页框物理地址 + PAGE_SHIFT

这个公式对前229376个页框都成立。所以，`__v()`和`__p()`能通过简单的加减`PAGE_SHIFT`就完成物理地址和线性地址的互相转换，而不用查页表。

大家发现，这229376个页框是在系统初始化时就已经被映射好的。回到问题，这就是为什么有的页分配过程中不用修改页表增加额外的映射：所有内核需要做的就是设置一个标志说明这个页框已经被分配出去了。

但对于另外一些页分配的过程，增加映射的过程是必不可少的——这就是高端内存的分配。在高端内存中进行页分配有两种方式：永久映射和临时映射。它们的区别简言之就是：永久映射每次都要在896~1024的线性空间找一个空闲的4K空间，然后在找一片排名229376开外的物理页框（排名229376前的已经被映射了），然后建立两者的映射。而临时映射采用固定预留的高端线性空间，多条内存控制路径分时复用这些线性地址空间——执行不同的路径需要修改页表项。临时映射没有有寻找空闲线性空间的过程，就减少了“找不到空闲线性空间”阻塞的风险。然而固定预留空间有限，所以不能同时大量建立临时映射。

为了完成永久映射的寻址，Linux引进了pkmap_page_table页表，每个页表项都可以用来映射4K高端物理内存区域。这满足了硬件分页电路寻址的需要，但是内核自己也想要寻址，需要像`__p()`以及`__v()`这样的简单方式，于是Linux又加了一个哈希表`address_htable`专门存放物理页框和线性地址的映射关系。

这里记录一个通用寻址函数`page_address()`。它可以通过一个页描述符，计算返回线性地址。之所以说它通用，是因为这个函数是`__v`和`address_htable`的组合，可以自动分辨页框属于正常区域、还是高端内存区域。

映射是建立物理地址和线性地址联系的过程。在文章开头，我们使用`kmap()`对一个普通区域的页框进行了映射。这个映射其实什么也没干，就是调用`page_address()`（实际上就是`__v()`）返回了线性地址。而对于高端内存，如果`page_address()`返回`NULL`，那就得为它修改或增加映射。

所以说，Linux中页的分配就是这样的一个过程：利用伙伴系统从`mem_map`描述的这一串内存区域中取得一个空闲物理页框（实际上还用了缓存来提高效率）。然后找一片线性空间并为线性空间和页框建立映射，返回这段空间的线性地址。