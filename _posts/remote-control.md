title: 在任何地方控制宿舍的电脑
date: 2016-04-04 17:41:09
tags:
---

三天假期出去浪，妈妈问我的代码是在哪里写的？

![](/img/remote-control-wild.jpg)

<!-- more -->

这是我一直的梦想，在哪里都能用上电脑！当然绝不是指购买十台八台电脑放在生活圈的各个角落，而是指随时随地访问宿舍里的电脑。

## 解决方案

问题就出在宿舍里的Dell处在内网，除了在一个子网的机器，别的任何机器都没法连接上它。

![](/img/remote-control-false.jpg)

解决的方法就是在互联网上找一台大家都能访问的机器做中转，把对宿舍电脑的操作命令先传递到中转机器上，然后由它发送到宿舍电脑上执行。这个中转的一种实现就是ssh的端口转发。这可不是一个新鲜的玩意儿。

![](/img/remote-control-true.jpg)

## 那具体怎么操作？

### 第一步，寻找一台中转Relay

对于中转的机器要满足要求是：拥有公网IP，换句话说，必须能在因特网上找到。

### 第二步，在宿舍的Dell上做反向端口转发:

	ssh -R 34345:localhost:22 <relay-usr>@<relay-address>

这行命令会将localhost(Dell)的22号端口映射到Relay的34345端口上。这样，我们对于Relay的34345的操作都会被Relay投射到Dell的22号端口上，例如：我们对Relay的34345号端口发出ssh连接请求，连接请求将会被转发到Dell的22端口上，所以我们实际在连接Dell，而这正是我们想要的！
	
### 第三步，修改中转机器配置

我们登陆Relay，检验转发是否成功:

	ssh <relay-usr>@<relay-addr>

根据提示输入密码，登陆Relay后，执行下面的命令：

	ssh  <dell-usr>@<relay-address> -p 34345

输入密码后我们登陆到Dell上。但操作还没有完全成功。这个端口转发只在relay机器上生效，还不能实现其它内网机器连接Dell。所以我们需要修改Relay上的ssh配置文件/etc/ssh/sshd_confg，打开GateawayPorts开关：

	# GatewayPorts no
	GatewayPorts yes

这样我们就可以在任意能连上因特网的机器上访问Dell了。

我们还可以尝试不同的ssh参数，获得一些功能。例如-f参数可以让转发在后台进行，不会因为当前会话的关闭而失效。

到这里文章就可以结束了。看我们的Dell，完美连接，流畅操作！……额，有一个问题，Dell得一直处在开机待命状态，这多浪费电呀、要排放多少CO2、多少小动物会因此丧生？为了避免Maxwellxly追来北京教导我保护环境，我决定加上一个远程电源控制。

## 远程电源管理

- IPMI
- phidgets
- http://www.digital-loggers.com/lpc.html

上面这些都是远程电源控制的解决方案，涉及到各种硬件设备。作为我，为了节约成本，我会时常想起那句话：

> 幸福来源于对已有物品的满足。

我有啥？一块cubieboard（一款卡片电脑，和树莓派是一类），我已经打它主意很久了。cubieboard功耗极低，待机一个月才一度电。我想用cubieboard一直处于在线状态，通过连接它去启动功耗较大的Dell。于是整个系统就成这样子了：

![](/img/remote-control-power.jpg)

那么cubieboard怎么去启动Dell呢？远程关机谁都会，问题出在把一台关着的电脑打开。想想半夜你床头的笔记本突然自己启动了……

好在现在的显卡都是支持wol（wake on lan, 网络唤醒）。虽然电脑是关着的，但是网卡还处在半死不活的状态，睁一只眼闭一只眼注意着网线上的信号。一旦它收到一串特定的字节，它就会把整台电脑唤醒。另外，wake on lan这个名字告诉我们，唤醒过程只存在于一个lan（局域网）里。很多路由器是不支持路由唤醒信号的。另外，最好使用有线局域网。

有了硬件的支持，我们看看怎么给网卡释放这个唤醒信号。Linux下有一个应用就叫wakeonlan。

安装

	apt-get install wakeonlan

使用

	wakeonlan <target-MAC>

哟嗬！电脑突然亮了，吓老子一跳！

好了，集齐了硬件、软件，剩下的就是把cubieboard也用第一部分介绍的方法暴露在因特网上，我们先连接它，让它唤醒Dell，接着再连接唤醒后的Dell。我打算再写一些脚本自动化一些过程，写一段[SystemV服务脚本](http://www.cyberciti.biz/tips/linux-write-sys-v-init-script-to-start-stop-service.html)开机启动......

但是我得停一下！写代码之前搜一遍Github是个好喜欢。运气不错，找到一款:Remote Wake/Sleep on LAN Server。它用web界面封装了wakeonlan和ping工具，可以实现唤醒和休眠(休眠只对windows机器有用)和机器状态查询，功能简洁干净！[使用说明点这里](https://github.com/sciguy14/Remote-Wake-Sleep-On-LAN-Server/wiki/Installation)

<center>

![](/img/remote-control-power-app1.jpg)  ![](/img/remote-control-power-app2.jpg)

</center>


## 等一下，你这个有点折腾！

简短的回答：因为我能！

其实实现移动办公有很多很方便、廉价的方法，例如直接使用云主机或者使用teamviewer软件即可。我迂回一大圈的原因很简单：因为我能折腾。除此之外还有一些个人原因：

公共租赁的云主机可是大大满足了我的需求，除了一点：尽管我可以按喜好配置它使用它，它的所有权不是我的。对于我这种控制欲很强的人来说，我是不会把重要资料和会常久使用的系统部署在一台听别人摆布的机器上的。

Teamviewer这类软件为我们提供了因特网范围内的远程桌面服务，但图形化的远程操控是卡顿的、耗费流量的。更关键的是，很多情况下远程桌面本身就是冗余的。

