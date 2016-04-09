title: 记录一次OSX软件破解
date: 2015-10-10 13:02:54
tags:
---
这不是一篇教你破解苹果软件的简明教程，而是一篇斗智斗勇的曲折故事。请支持正版！

<!-- more -->

# 序章：惹错人了

听说精致的Mac和优雅的MarkDown很配。你知道Markdown也可以用来写作幻灯片吗？首先给大家推荐一个Mac平台上优雅的应用叫做DeckSet。售价29.99$，挺贵的，呵呵。

我真的是一个自由软件爱好者！我虽然穷我支持正版！但我同时是个愤青啊！AppStore慢也就算了，想注册个可以花钱的账号过程也太不友好了！关键是，安装完DeckSet你跟我说公式渲染要额外付费9.99$是几个意思？

而且我仔细观察软件的行为，发现它的公式解析使用的是MathJax——一款开源免费的程序。DeckSet从开源软件中免费获得了代码，转而将这些代码实现的功能通过收费的形式提供给消费者，太不人性了（不过这不构成侵权，因为MathJax遵循的Apache协议允许商业销售）。

如果你不付钱呢，它就给你这样弹警告：

![](/img/extra-money.png)

给你这样(把你的公式给吞了，仔细看还有淡淡的影子):

![](/img/no-formula.png)

不管怎么样，我想它是惹错人了。

# 利其器

我并不是一个职业黑客。我之前也没破解过软件。既然放了狠话，也只能硬着头皮上。看看我已经有啥技能和工具：掌握C语言，有过软件设计的些许经验。由于踩过无数坑，渐渐总结出了软件调试技巧。加上一些新手的运气和对奸商的憎恶——可以付之一搏了。

再看看手头工具有哪些呢？静态观察程序代码，那就得要反汇编器，我觉得otx就不错。动态观察程序运行，那就得要调试器，就用Xcode自带的gdb吧。编辑二进制文件，系统自带的vim就可以。都是一些没有图形界面的土装备，但够用就行，烦不了太多——我是新手我怕谁。

otx用homebrew就可以安装。前提是必须先安装好Xcode。

gdb据说是Xcode自带。不过之前我用homebrew大法安装了独立版本。受限于系统权限，gdb想调试一个程序并不容易。你得用证书把gdb签名，赋予它调试其它进程的权限。具体过程可以参考[这个](http://www.csdn123.com/html/topnews201408/43/8443.htm)。

# 了解你的敌人

为了破解一个软件，我们不得不先了解这个软件。为了了解我们的目标，我们得搜集它的各种信息。

搜集信息的第一个层面是你要知道敌人在哪？我是说你的知道软件的安装位置和目录结构。Mac系统上，软件被装在/Application/目录下。其中/Applications/XXX.app/Contents/MacOS/这个目录下就是程序的二进制文件——我们靶子！

搜集信息的第二个层面就是多用这个软件，对与一些提示要特别留意。比如第一幅图中那烦人的黄色警告条："Buy formula support for $9.99"就变成了我的突破口。

首先我用下面命令对程序进行反汇编，汇编代码保存在deckset.asm文件里。

	otx Applications/Deckset.app/Contents/MacOS/Deckset > deckset.asm

打开保存结果的文件，里面全是汇编代码和各种零零碎碎的信息。我是新手，我看不懂。我不管，先Ctrl+F查找"Buy formula support"。果然，这样这样函数，对某个值进行了判断，并跳转到输出"Buy formula support"。跟踪这一线索，不说能不能了解程序判断你是否是合法用户的机制，倒是眼前肯定闪过了不少能让你想入非非的词汇，比如：

- initWithMASProduct(MAS是不是maths?Product！)
- RecieptVerification(Verification?验证什么？)
- systemMACAddress(是不是用了MAC地址进行验证啊？)
- purchaseFormulas

这个时候一些软件设计的经验就派上用场啦！用猫的思维去思考的老鼠活得才自在，用老鼠思维去思考的猫才能抓到最多的老鼠。知道怎么拼软件才知道怎么拆软件。

这时google到了一个工具叫class-dump（听到dump就来劲！），可以从面相对象编程的OC二进制文件中提取类的信息。这就是搜集信息的第三个层面，程序代码中蕴含的信息。otx反汇编时其实已经包含零零碎碎的信息啦，只不过class-dump的信息是按面向对象的思想整理过的。

otx也好，class-dump也好，不管用什么工具，意识最重要。刚刚看过的那些想入非非的字符串的最后一条“purchaseFormulas”激发了我的灵感。我Ctrl+F用purchase一搜，得到了下图信息：左边是otx反汇编的代码，右边是class-dump的结果，可以验证这些信息两个工具都是可以提供的，只是class-dump更便于阅读）:

![](/img/purchase.png)

看到这个函数名"purchasedAddOnWithIdentifier"，加上返回值类型是BOOL，以及一些想当然，可以说任务已经完成一大半了。明白了吗？这个函数很可能是用来验证用户是否购买过这个AddOn，如果买过就返回True，否则False。所以我们只要让这个函数一值返回True很可能就大功告成啦！

# 准备毒药

来，我们仔细看purchasedAddOnWithIdentifier代码！

![](/img/purchase_code.png)

你真的仔细看了？我开玩笑的:P 我自己都看不懂！不需要看懂，我就是想让这个函数一直返回1而已。大神可能已经在用大脑编译一段实现这样功能代码然后直接用机器码填上了——这我可做不到，但我机灵着呐：我用C语言写一段一直返回1的函数，然后编译这段C代码，接着从里面提取这段机器码。

好，现在就开始做！写一段熟悉的C语言代码：

	#include <stdio.h>
	
	int freeman()
	{
		return 1;
	}

	int main()
	{
		if (freeman())
			printf("hello");
		return 0;
	}
	
编译完了在用otx反汇编，得到：

![](/img/alway_true.png)

用图中高亮显示的8字节机器码替换掉purchasedAddOnWithIdentifier开头的代码，就可以让purchasedAddOnWithIdentifier函数永远返回1，也就是True啦。这8字节机器码就是我们的毒药，下面我们来看看怎么把毒药喂给程序。

# 下毒
明确任务，我们需要把图左边的8字节代码一一换成右边对应的代码。

![](/img/paper.png)

但是还记得吗？我们上述操作都是建立在purchasedAddOnWithIdentifier是关键验证函数这个猜想上的，我可不想在可执行文件本身直接动手，而是选择在运行时内存镜像中进行动态hack。这里就用上了gdb和一些特殊的调试技巧。

	gdb deskset
	> b *0x00000001000c43b2   #purchasedAddOnWithIdentifier函数开始地址
	> r
	
然后随便摆弄一下程序，直到其“卡”在断点上。谢天谢地，程序真的停下来了！然后开始用gdb的`set `命令修改`0x1000c43b2~0x1000c43be`范围内内存单元的值。set完了，再用`disas 开始地址，+区间长度`命令反汇编这一段内存区域，检查是否正确。从下图可以看到，我们的代码和C语言hello程序中的freeman函数反汇编的结果是一样的。

![](/img/set_disas.png)

然后 `continue` 运行。成功了！

清爽的界面再没有警告！购买菜单也灰掉了。

![](/img/success.png)


最重要的是公式终于清楚地显示啦！

![](/img/success2.png)

# 稳固胜利果实

试验成功了，我们的毒已经注入了程序的内存空间。但是这个破解可不是永久的，每次重新打开程序，就得重复上述步骤进行破解。现在我们要把胜利的果实固定住。这可以通过修改程序二进制文件进行。用到的工具就是vim。

利用vim打开二进制程序看起来不是一个好主意——一片乱码啊！不过在vim中敲`:`然后在下面输入`%!xxd`回车后看到了可是另一番景象。通过搜索匹配到我们的目标代码，然后逐一替换，保存即可。

![](/img/hex.png)

具体操作时有一些问题，比如说目标代码不好定位。这里提供一个小的脚本工具：[offset](http://reverse.put.as/wp-content/uploads/2011/02/offset1.3.pl_.gz)用来定位。

替换完成后别急着`wq`保存退出。而是先`%!xxd -r`重新转码然后再保存退出。

这样，我们就可以优雅地运行程序了。

哦，对了请支持正版！


# 参考资料

- https://reverse.put.as/wp-content/uploads/2011/02/beginners-tut-II.txt

- http://kswizz.com/2011-01-16/hacking-mac-apps/

