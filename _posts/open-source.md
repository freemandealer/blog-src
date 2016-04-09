title: 开源工具的使用
date: 2014-5-6 17:03:04
tags: Note
---

<!--more-->

##1 MarkDown和GitHub的简介
###1.1 MarkDown
MarkDown是一种标记语言，通常被用来作为开源图书、博客的写作工具。不同于Microsoft Word，MarkDown可以让写作者不用过多地去担心排版而集中精力去完成内容的写作，并且记号简洁明了，效果美观大方。
可以看一个[例子](#markdown_eg)，了解MarkDown的语法。
###1.2 Git/GitHub
Git是一个分布式的版本控制系统，最初由Linus Torvalds编写。通常用于版本控制和多人合作。使用Git需要有一个服务器，GitHub就相当于一台互联网上的公共服务器。

GitHub的术语及其概念:

- 远程公共仓库/远程个人仓库/本地个人仓库
- fork:从远程公共仓库克隆到远程个人仓库
- pull:从远程个人仓库下载到本地个人仓库
- push:从本地个人仓库更新到远程个人仓库
- commit(提交):对下载到本地的项目进行快照,可以理解成"存档"
- 提交信息:提交时需要填写的信息，如：提交时做了哪些修改

##2 在我们的项目中使用MarkDown和GitHub
###2.1 前期准备

- 下载安装Git Bash [https://code.google.com/p/msysgit/downloads/list](https://code.google.com/p/msysgit/downloads/list)

- 需要注册一个GitHub账号 [www.github.com](www.github.com)

- 登陆项目主页 [https://github.com/freemandealer/javaee-together-we-will-learn](https://github.com/freemandealer/javaee-together-we-will-learn)并Fork该项目到自己的仓库。现在在你自己的Repository中应该有了项目的副本。


- 下面把项目副本下载到本机，大家就可以开始图书的编写与修改了，具体做法:
 
    1. 在本机中新建一个文件夹，打开文件夹，单击右键，选择`Git Init Here`以初始化目录。
    2. 单击右键，选择`Git Bash`，打开命令行。
    3. 使用命令下载代码到本机。
    
    `git pull https://github.com/你的用户名/javeee-together-we-will-learn.git master`
###2.2 在平常写作中使用Git
我们初始化了Git，后面就可以对下载到本机的工程进行修改、存档和提交了：

- 使你的工程和大家保持一致，(永远的第一步!)

    `git pull https://github.com/freemandealer/javaee-together-we-will-learn.git master`
- 做一系列的编写和修改
- 使用`commit`进行"存档"。用法：

    `git commit -am "提交消息"`
- 提交到你自己的远程仓库
    
    `git push https://github.com/你的用户名/javaee-together-we-will-learn.git master`
- 在GitHub网页上提出申请，使你的改动能够进入公共仓库。

###2.3 MarkDown的格式约定
在本开源图书编写过程中，为统一格式，我们做如下约定：

仅使用三级标题，即一级标题(用于章)、二级标题(用于节)、三级标题(用于小节)。举例如下：
#第X章 
##1.1 JavaEE概述
###1.1.1JavaEE的发展过程
JavaEE作为当今Web技术...
###1.1.2JavaEE的特性
...

##附录

<span id="markdown_eg">
###一个MarkDown例子
	#一级标题
	##二级标题
	内容写在这，**粗体**，*斜体*
	
	段落之间空一行
	
	这个是嵌在文字间的代码片段`<html>`，下面是独立的代码段
	
	    <html>
	        <body>
	            hello!
	        </body> 
		</html>
	插入图片![图片标题](C:\0_example.png)
	
	[超链接名字](www.超链接地址.com)

#一级标题
##二级标题
内容写在这，**粗体**，*斜体*

段落之间空一行

这个是嵌在文字间的代码片段`<html>`，下面是独立的代码段

    <html>
        <body>
            hello!
        </body> 
	</html>
插入图片![图片标题](C:\0_example.png)

[超链接名字](www.超链接地址.com)

</span>
