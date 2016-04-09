title: 内核调试工具SystemTap:适合懒人的printk替代品
date: 2015-04-13 17:14:23
tags: Note
---
SystemTap是一个Linux调试和性能分析工具，可用于应用层和内核层的分析，但主要侧重内核层。SystemTab可以在**不修改内核代码**、**不重复编译内核**、**不重启机器**的情况下，收集运行内核的信息并使信息可视化。调试人员可以利用它绘制函数调用关系图，打印寄存器信息和调用栈，输出内核中指定变量（可以是局部变量）。它如同一个更加方便prink，方便调试人员观察内核行为，诊断错误异常，分析系统性能。在YAVIS开发过程中，我们使用SystemTap分析包发送和接收情况，并分析通信性能瓶颈。

<!--more-->

## SystemTap工作流程 ##
SystemTap使用了Kprobe技术探测内核信息，辅以Relayfs向用户传递消息。

SystemTap首先将SystemTap脚本文件翻译成C源文件。这个C源文件实际上是一个内核模块，实现了脚本文件中描述的功能。接着SystemTap编译源文件获得二进制模块文件，并动态加载模块。模块被载入运行内核后，会报告脚本文件中指定的一些事件。事件会触发脚本文件中编写的处理函数，执行相关操作。一般操作内容是：收集所需信息，并通过标准输出打印给用户。SystemTap会话结束于用户发出中断信号，即`Ctrl + C`，内核模块将随之被安全卸载。

![](/img/systemtap.png)

SystemTap提供了一些内置函数，帮助我们快速开发测试脚本。常用的内置函数如：

- print(str) - 打印str的值
- printf(fmt) - 如同C语言的printf函数
- probefunc() - 返回当前探测函数的函数名
- execname() - 返回当前进程的名字
- pid() - 返回当前进程ID
- uid() - 返回当前进程用户ID
- cpu() - 返回运行当前进程的CPU号

另外一些内置功能以Tapset的形式出现。Tapset相当于SystemTap的库。它提供的功能不仅仅是函数，还包括一些预定义的探测点,如：

- timer.ms(N) - 每N毫秒探测一次（用于性能测试）
- begin - 探测模块加载时执行一次

当然，用户也可以开发自己的Tapset。

## SystemTap的使用 ##
在CentOS中SystemTap可以用下面命令安装升级：

    sudo yum install systemtap systemtap-runtime

编写脚本后，使用这个命令执行：

    stap <script-name>
    
注意：如果探测的是模块，要确保模块被复制到`/lib/modules/<kernel-version>/`中，否则脚本解析时会在`module`处出错。

## SystemTap脚本语言 ##
SystemTap语言是一种与C语言和awk语言类似的脚本语言。限于篇幅，这里并不系统地介绍SystemTap语言，而是结合毕业设计的调试场景，使用例子说明SystemTap的语法特征和编程结构。

    #!/usr/bin/stap
    probe module("yavis").function("*").call {
            printf("%s -> %s\n", thread_indent(1), probefunc())
    }
    probe module("yavis").function("*").return {
            printf("%s -> %s\n", thread_indent(-1), probefunc());
    }

上述代码的功能是输出YAVIS的代码调用关系图。第一行描述脚本采用的解释器是stap程序。第二行表示在yavis模块中所有函数中插入探针，并在这些函数调用时触发第三行的代码。第三行代码向标准输出打印一串信息，信息包括当前函数的名字，由内置的probefunc收集。第五至第七行代码与上面的类似，只是在函数返回时触发。这样，所有YAVIS模块的函数在调用时输出函数名，返回时再次输出函数名，同时由内置的thread_indent函数负责自动的缩进，最终绘制了整个YAVIS模块的函数调用关系图。

    #!/usr/bin/stap
    probe module("yavis").function("yavis_poll") {
    	   if ($revt->type == 0) {
    		  printf("-- package received --\n")
    		  printf("revt.msg_len = %d\n", $revt->msg_len)
    		  for (i = 0; i < $revt->msg_len; i++) {
    			 printf("%x", $revt->rbuff[i])
    		  }
    		  printf("\n")
        } else if ($sevt->type == 0) {
    		  printf("-- package sent --\n");
    	   }
    }
    
    probe module("yavis").function("yavis_tx") {
    	   printf("-- sending package --\n");
    	   printf("skb->len = %d\n", $skb->len)
    	   for (j = 0; j < $skb->len; j++) {
    	       printf("%x", $skb->data[j])
    	   }
    	   printf("\n")
    }

上面这段脚本可以让我们在发送和接收过程中查看数据。注意结构体类型的数据，无论指针与否，一律使用`->`方式访问成员变量。

## 参考 ##
[1] SystemTap: Instrumenting the Linux Kernel for Analyzing Performance and Functional Problems. IBM, 2009.
