title: 内核补丁的创建和发送过程
date: 2015-07-05 22:00:00
tags: Note
---


**1 确认补丁基点**
在制作补丁前，确认基点，并提交一次补丁前的状态。

<!--more-->

**2 创建补丁分支**

**3 做出修改，并提交修改**

提交时使用`-s`参数进行签名：

	git commit -s -m "fix something"

**4 生成补丁**

	git format-patch -<number>

**5 检查补丁格式**

	perl checkpatch --no-tree --strict  <patch-name>

**6 配置git-email**
通过git config配置邮件相关信息。修改的实际上是~/.gitconfig文件。

	git config --global user.name "Freeman Zhang"
	git config --global user.email freeman.zhang1992@live.com
	git config --global sendemail.smtpserver smtp.live.com
	git config --global sendemail.smtpuser freeman.zhang1992@live.com
	git config --global sendemail.smtpserverport 587
	git config --global sendemail.smtpencryption tls

也可以跳过这一步骤，通过git send-email参数在发送时确定邮件服务器信息。

**7 发送补丁**

可以编辑补丁文件的Subject等信息。
补丁可以连续依次发送，也可以放在一个目录下，发送整个目录。
`--chain-reply-to`参数将导致后一个补丁邮件作为前一个邮件的回复。对应`--no-chain-reply-to`会构造一个平坦的结构——这也是邮件列表要求的patch发送方式:

	git send-email --compose --no-chain-reply-to --suppress-from --to <target audiance> <patches>

下面弹出compose界面，这里有个'in reply to:'可以用来填写一个想与之构成thread chain的之前发送过去的邮件的MessageID(如果使用的是Thunderbird客户端，可以通过ViewSource查看邮件的MessageID)。举个例子：你先发送了一份cover letter（无论用什么客户端，只要plaintext就行）叫做’XXX CoverLetter‘，然后你的后续补丁就应该使用上文提到的`--chain-reply-to`将补丁们挨个串起来。最后填写这个`in reply to:`来将补丁与cover letter串起来。这样就构成了一个如下有深度的结构。

	XXX CoverLetter
	└─> [PATCH 1/2] XXX1
		└─> [PATCH 2/2] XXX2

但邮件列表建议我们使用`--no-chain-reply-to`方式，结果更加平坦便于审阅：

	XXX CoverLetter
		├─> [PATCH 1/2] XXX1
		└─> [PATCH 2/2] XXX2



** 可选 **
使用esmpt传输客户端发送邮件和补丁。
	sudo apt-get install esmtp
	touch ~/.esmtprc
	chmod g-rwx ~/.esmtprc
	chmod o-rwx ~/.esmtprc

为esmtprc加上：

	identity "my.email@gmail.com"
	hostname smtp.gmail.com:587
	username "my.email@gmail.com"
	password "ThisIsNotARealPassWord"
	starttls required 

使用命令行下的邮件客户端mutt处理PlainText邮件。

	sudo apt-get install mutt
	vi ~/.muttrc

为.muttrc加上：

	set sendmail="/usr/bin/esmtp"
	set envelope_from=yes
	set from="Your Name <my.email@gmail.com>"
	set use_from=yes
	set edit_headers=yes

输入`m`创建新邮件，保存后`y`发送，`e`继续编辑，`q`放弃发送并退出。
