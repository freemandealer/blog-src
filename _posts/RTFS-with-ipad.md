title: 如何优雅地使用iPad阅读源代码
date: 2015-07-30 14:10:50
tags: Note
---

> "Read the fucking source code! "
> -- Linus Torvalds

如何优雅地使用iPad阅读源代码？答案很简单呐！只要花50大洋买一套国人开发的CodeNavigator付费版就好了。作为代码阅读工具至少有两个handy的功能吧：语法高亮和代码跳转。CodeNavigator提供的注释功能可以写写笔记也很贴心！并且，CodeNavigator还支持git的整合（反正能想到的功能和不需要的功能人家都做好了）。作为比较，AppStrore里我所检索到的其他工具，大部分仅支持语法高亮。

<!--more-->

本博客到此结束，结论就是：想优雅地在iPad上阅读代码，那就去购买CodeNavigator吧。下面进入问答环节：

** 1) 50块好贵...人家只想看小工程而已... **
对于个位数的源文件、每个文件几百行的小工程，看代码其实只要高亮功能就好了，随便去AppStore下个免费的代码编辑器就好了。

** 2) 我没钱！我要看_大_工程！ **
如果你读的是Linux内核代码，那么使用Safari打开一个LXR(Linux Cross Reference)网站就可以在线阅读各个版本的Linux内核代码了。

[lxr.free-electrons.com](http://lxr.free-electrons.com)
[www.cs.fsu.edu](http://www.cs.fsu.edu/~baker/devices/xref.html)
[lxr.oss.org.cn](http://lxr.oss.org.cn)


** 3) 我没钱！我看代码的地方没网！我还是要看_大_工程 ! **
那你得有台树莓派，自己架设一个LXR服务器随身带着。

** 4) 我没钱！我没网！我没树莓派！我依然要看_大_工程！ **
那...那你得越狱iPad装破解软件。

** 5) 我没钱！我没网！我没树莓派！而且我不能将就使用盗版软件！我依然要看_大_工程！ **

你！！你这个性格很像我啊。你这种情况也能解决，但前提是你爱好折腾。

现在我们就用开源软件和免费软件为原材料，自己动手做一个CodeNavigator，就叫做HodeNavigator　(HOme-maDE Navigator) 。其实就是个山寨版，名字不用太洋气，就这样吧～！

咱们的HodeNavigator需要解决的问题是：它需要实现语法高亮以及最重要的，代码跳转的功能———用行话说是交叉引用(Cross Reference)———而且不能借助于服务器（毕竟我们没网没设备什么都没有嘛）。问题复杂但我的思路很简单：将要观摩的代码生成一个支持跳转的文件，然后放到iPad里打开不就行了？（不要打我）

这样支持导航和跳转的文件类型我知道的有:HTML和PDF。那到底选择谁？我淘汰了PDF选项，因为我并没有想到从源代码生成PDF文件的好办法。用于生成文档的Doxy也许是个办法，可是我还没有学会设定配置文件以产出纯粹的交叉引用代码，换言之就是生成的文档很杂，尺寸大到ipad爆炸！不过PDF的好处是使用iPad自带的iBook就可以轻易打开，如果看官们有好的生成办法一定要留言分享啊！这里我们选择使用HTML。

这是开源界为我们提供的交叉应用服务工具：

[Comparison with Similar Tools](https://github.com/OpenGrok/OpenGrok/wiki/Comparison-with-Similar-Tools)
	
这么多，我们选谁？由于我们生成的是静态HTML文件(动态的就牵涉到使用服务器了)，我们看`Static HTML`那个指标，就知道GNU的Global软件是我们的唯一选择。

通过你自己喜好的方法安装好global之后，我们进入到需要观摩的源代码目录并执行：

	gtags && htags
	
经过一段时间等待后，我们就可以在当前目录下找到`HTML`文件夹，打开里面的`index.html`，你会发现我们的交叉引用生成好了！

可是问题又来了，这个交叉引用不是一个单一文件，而是目录（中所有文件组成的）。我们怎么在非越狱的iPad中打开一个目录呢？

答案是，我们可以用工具整合这个目录变成单一的文件。我这里选择使用Windows下的QuickCHM工具对整个目录进行编译，编译后得到了单一文件。该文件格式为`*.chm`，是Microsoft的一种古老的帮助文件。一个额外的奖励是：文件变小了！当然你可以采取任何你喜欢的方式得到这个单一文件，比如使用其它工具(linux底下也有chm编译工具，被粗暴地命名为chmc，但目前版本的chmc并不能胜任如此复杂的编译)，甚至编译出其它格式——前提是iPad下有对应工具可以打开这种格式的文件。

对于`*.chm`格式，AppStore里可有不少能阅读它的App，可以根据个人喜好选择。我选择了一款没有广告闪瞎我双眼的App叫"CHM Sharp"。

通过百度云或其他方式把制作好的chm文件传到iPad上打开，就可以在像金陵图书馆这样没有WIFI、饭菜难吃、公共自行车站设置到地图边界以外的地方，按亮pad，翻开*Understanding the Linux Kernel*，感受“啃核”的激情了！

** 6) 我没钱！我没网！我没树莓派！我不能将就使用盗版软件！我还没iPad！ **
那就放弃使用iPad, 这个点子很是easy adopting. 尝试在其它平台实现这个想法，回头告诉我你的创意吧 :)

![截图](/img/rtfsc-ipad.png)

**福利！** 附制作好的Linux2.6内核阅读文件(52MB):
[CSDN资源](http://download.csdn.net/detail/freemandealer/8948651)
[百度盘资源](http://pan.baidu.com/s/1hqm1sPm)