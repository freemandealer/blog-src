title: Windows环境下Hexo博客的搭建
date: 2014-11-27 21:49:47
tags: Note
---
> 这篇文章很长，因为它很简单^_^。

<!--more-->

## 安装环境

为了使用Hexo，我们需要安装下面的软件：

- msysgit:git工具，这里用来上传和下载文件
- node.js:运行环境
- npm：node.js用来安装其他包的工具

为了方便大家，我将上述安装文件打包上传至百度盘，[点击下载](http://pan.baidu.com/s/1bnu8Xwr)。

保证后面的工作能顺利进行，在安装msysgit出现下面画面是请选择中间那项：

保证后面的工作能顺利进行，在安装msysgit出现下面画面是请选择中间那项：

![msysgit安装](/img/msysgit.png)

最后，按住`ctrl+x`(或采用其它方式)，用管理员身份运行命令提示符。运行命令安装hexo:

	npm install -g hexo
	
![hexo安装](/img/hexo-setup.png)

如果出现错误，很可能是因为命令提示符没有运行在管理员状态。

## 尝试Hexo

安装好环境后，我们先来运行博客感受一下。

首先在适当的位置建立一个文件夹存放博客的内容。如E:\blog\。

使用命令行进入博客目录，如：

	C:\Users\Ryan>E:
	E:\>cd blog
	E:\blog>
	
运行下面的命令初始化博客

	hexo init
	
上述命令得到提示：

	[info] You are almost done! Don't forget to run `npm install` before you start blogging with Hexo!

根据提示我们运行：

	npm install

接着执行：

	hexo s

提示：

	[info] Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.

打开浏览器，输入网址`http://localhost:4000/`便可浏览hexo默认的博客。可能由于Google字体的原因，页面加载会卡住，这时候需要停止加载页面才能看到网页。

## 配置博客

看到了默认博客，说明之前我们的工作做得很出色，已经成功了一大半。现在我们正式开始自定义博客，并让它发布到互联网上。

### GitHub

我们用[GitHub](www.github.com)作为我们发布博客的容器。进入GitHub并注册一个账户。

接着在网页上进行一些操作，创建一个新的repository。注意这个repository的名字必须是: `你的github用户名.github.io`。

### 配置Hexo
 
在博客的目录里有个名为_config.yml的文件，就是Hexo的配置文件。这是一个文本文件，但是建议大家不要用记事本或写字板打开，因为格式或字符编码会带来大的问题。推荐使用Notepad++：

	...

	# Site
	title: Season's Blog    <--修改这里可以改变标题栏
	subtitle:                 
	description:
	author: Season Lee        <--修改这里可以改变网页页脚的作者名字
	email:
	language:
	...

	# Deployment
	## Docs: http://hexo.io/docs/deployment.html
	deploy:
 	   type: github        <--输入github,然后增加下面两行
 	   repository: https://github.com/你的github用户名/你的github用户名.github.io.git
 	   branch: master
 	   
根据提示修改值。变量名都是自解释的。

## 平常使用
配置好博客，我们就可以使用了。

hexo 常用命令如下：

- hexo g: 生成——每次修改或增加内容时运行一遍
- hexo s: 本地预览
- hexo d: 发布到网上

如果不想使用命令行，可以下载我写好的小工具包，解压到博客所在目录。工具的名字是自解释的。 [点击这里](http://pan.baidu.com/s/1qWt26fU)下载小工具。

第一次发布，可能提示:

	Initialized empty Git repository in E:/blog/.deploy/.git/
	*** Please tell me who you are.
	Run
 	git config --global user.email "you@example.com"
 	git config --global user.name "Your Name"

则请按照说明，执行：

	git config --global user.email "you@example.com"
 	git config --global user.name "Your Name"

发布时需要提供 github 的用户名和密码。密码在输入过程中是不会显示的，觉得自己输对了回车就可以了。

在浏览器中输入`你的github用户名.github.io`便可以访问博客。快把链接发给你的朋友们吧~

第一次发布，需要等一段时间才能看到自己的页面（20分钟内你只能看到404找不到页面）。除了第一次，以后的发布会立马显示结果。

## 还能做什么
Hexo很灵活，通过插件和其它配置选项，我们可以实现丰富多彩的功能。如果你愿意付少量的费用，还可以购买个性域名。在这里，我简单介绍主题的变更，并为朋友们评论自己的博文增加一个留言模块。

### 更换主题
推荐一个地址，大家可以在这里集中寻找自己喜欢的主题。 [点击这里](https://github.com/hexojs/hexo/wiki/Themes)

点击某个主题，基本上每个主题都有一个Demo可以预览。同时会看到关于这个主题的安装方式。主题安装方法大致相似，并且非常简单：

- 使用命令行进入博客目录，如：

	C:\Users\Ryan>E:
	E:\>cd blog
	E:\blog>

- 执行下面的命令,具体参看每个主题的安装指南(Install):

	git clone git://github.com/XXX/YYY.git themes/ZZZ 

- 修改_config.yml

		# Extensions
		## Plugins: https://github.com/hexojs/hexo/wiki/Plugins
		## Themes: https://github.com/hexojs/hexo/wiki/Themes
		theme: landscape   <-- 改成主题名字ZZZ
		exclude_generator:

- 预览并发布

### 增加留言功能

使用’多说’插件，增加留言功能。

首先，需要注册多说的账号。[点击这里](http://duoshuo.com/)。注册完会得到一串通用代码。

接着在博客目录下的`theme/你现在在用的主题/layout/_partial/`里/附近（每款主题不一样）找到`comment.ejs`文件，用注册得到的通用代码替换原有内容。

修改`theme/你现在在用的主题/`下的配置文件_config.yml(是主题的配置文件而不是博客的配置文件)。修改一下内容：

	# Duoshuo comment
	duoshuo:
  	enable: true        <-- 如果没有enbale就增加一行
  	short_name: Season    <-- 这里填写"多说"的用户名

预览一下吧。

## Q&A
博客搭建过程中出现了很多错误，记录在这里希望能帮到别人。

使用方法：Markdown，有中文语法解释。[Markdown中文语法解释](http://wowubuntu.com/markdown/index.html)

安装出现问题： 你是用管理员身份使用命令提示符的吗？

发布以后等了好久，一直是404错误提示找不到页面：你的github账号通过邮箱验证了吗？

预览时网页上显示了很多代码： hexo init 后是不是忘了 npm install？

生成时说配置文件有错误： 配置文件冒号后面漏空格了吗？ 另外，deploy那一段是有缩进的。

写文章怎么引用图片？在source文件夹下新建img文件夹放所有文章的图片，在文章中使用`![图片标题](/img/图片名.图片格式)`引用之。

## 写在后面

像一名黑客那样写博客。于是博客似乎成为了黑客和Linux高手的专属玩具。很多人并不是计算机方面的精英，但是这仍然不能阻碍他们成为自己领域的黑客高手。随着工具的发展，搭建博客的技术门槛越来越低。这篇文章，写给Windows平台上的普通用户，只要你热爱分享，热爱生活。
