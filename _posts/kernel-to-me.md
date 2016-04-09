title: 内核之于我
date: 2016-02-02 22:29:20
tags: Notes
---


今天导师发来邮件说让我去做无人汽车的人工智能，涉及深度学习、人工神经网络等一系列时髦词汇。很多人都觉得着实刺激，称赞是一个不错的课题。但我心里咯噔了一下——我亲爱的内核怎么办？

<!-- more -->

这个突然出现的深度学习课题，一旦参与，很大可能研究生未来的日子里都是跟它一起过日子了。进行了三年的内核研究似乎不得不停止，摆弄操作系统沦为不务正业。回想大二下学期稚气的我偶遇稚气的《30天自制操作系统》，想想大三冬夜熄灯后坐在楼道里啃《一个操作系统的实现》，想想每逢寒假暑假在金陵图书馆死磕内核代码，想想那本走到哪带到哪的大部头《深入理解Linux内核》…… 有时侯室友张刚都评价我的做事方式：你都把操作系统的思想带入生活了。真是分手时才想起她的好！

煽情部分到此结束，我们下面来看看内核的经历到底给予了我什么（喂～）

当然首先是钱途！在刚上大学时，就和一位名校毕业的在职驱动工程师交流过。驱动也是内核编程的一种。真的不能小看前辈（好吧，其实是名校毕业生）话语的力量，我的很多选择追根溯源也许就是受这次简单交流的影响。他让我感觉内核编程是复杂的，如果我能搞定这个，其它的编程不是问题。另外就是驱动程序猿的工资会高出很多。反正搞内核就是很高端的样子，不管是想证明自己，还是为了虚荣心，我渐渐不屑曾经执着过的酷炫前端和手机应用开发。

不过坑爹的是，后来他告诉我，国内的驱动开发其实也是写事务性的代码，重复度很高，工资也并不乐观。而我现在也混进名校了好嘛！除了做驱动开发，那就是真正的内核开发了。具体内核开发是什么，这正是我**正在**探索的。

除此之外，内核还给予了我什么？满足好奇！是一个十分有重量的理由。我很好奇计算机系统（广义的系统，包含硬件和软件）的如何工作，为什么这样设计。内核是软硬件的交界面，是理解整个计算机系统的切入点。况且，内核的学习资源很丰富，有很多著名书籍讨论其理论，又有像Linux内核开放源代码。这些来之不易的资源让我们相信：只要愿意下功夫，内核毫无秘密。

遗憾的是，随着我对内核的理解，这股好奇的力量逐渐减弱，这时内核之于我的其它意义凸显出来：内核赋予了我把握复杂软件系统的能力。Linux是开源的，是伟大的。Robert Love说：“请不要认为这理所应当。”而我还要加上一句：“请不要认为有了代码就没有了秘密。”沉浸在上亿行代码中，可不是一种愉悦的感受，而是分分钟溺亡的窒息感。当学会了拆分系统，学会了观察目录组织，学会了猜测函数功能，培养了寻找主干的嗅觉，培养了抽丝拨茧的侦探精神等等一系列特意功能后(好吧，其实只是耐心被逼出新境界了），再看各种大型工程，不会觉得心里烦躁了。

内核编程本身就是一项实用技术。很多应用层编程完成不了的功能，放到内核层实现就能成为可能并且会很自然很优雅。计算机网络课程大作业，我也看到很多小组不约而同地使用了netfilter框架，这也是内核级的编程。从这点来看，内核编程和和IOS开发一样，都是在一定平台下，实现了一些功能。不过内核编程的平台直接是硬件，在这个混沌的世界编程更加复杂。有了内核基础的我面对这些任务当然是毫不畏惧，相反，游刃有余。

Linux作为世界上最大开源项目，应该是没有之一的。作为内核开发轻度参与者，或多或少也受大胡子理念熏陶，学着用起来Git和GPL神器，和大牛们发起了邮件，感受到了“社区的力量”。

研究内核训练了我解决问题的能力。还记得大三做内核相关的项目，及其艰深，学校老师没有人懂，身边也没有经验的同学，完全就是一个人在那看时光荏苒思前想后。然而结果证明这个孤立无援的环境成为了我的成长的茂盛丛林。从开始学习，尝试读不同的书，最后理清书本间的关系找到一条斜率适度的学习曲线。从项目思路到设计到编码到debug，凡事出现问题，都是一个人使出浑身解数，丧心病狂。值得一提的是最终我结识了散落在各地的小伙伴、找到了组织(找到组织本身就是一个人死磕出的结果呀！)，又开始学习通过交流和他人的经验解决问题。想想内核debug的悲惨遭遇，还有什么算问题呢？

说到这里，让我们回到开头时的困惑：选择参与了实验室项目，很可能就得放弃内核。实验室课题、就业和个人兴趣这个三角很难被平衡。有人就说了：干嘛平衡？搞得像以后工作内容和你现在弄的东西相关似的。听完我虎躯一震：潇洒！

上次同样的一震还是谢高岗老师的复习课上，我们满心期待划重点，结果PPT上画着一个扫地僧，伴有旁白：手中无剑心中有剑，无剑亦为有剑。如果一个人把自己学到的具体知识忘光，那么剩下的就是理解和能力。如果我以后没有和内核走在一起，忘记了她，那么剩下的，就是文章中的这些沉淀吧。


![](http://i13.tietuku.com/8acb1f0185975269.jpg)