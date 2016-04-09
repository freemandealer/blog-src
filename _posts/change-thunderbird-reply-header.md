title: 修改Thunderbird的回复头语言和样式
date: 2014-11-28 11:16:59
tags: Note
---
当你回复一封邮件时，Thunder会自动增加一个类似这样的回复头(reply header):

<!--more-->

    在2014年11月28日 1-:32, XXX写到：

当我们回复英文邮件时，这句默认添加的中文就显得有些不适合了。那么我们怎么才能修改回复头的样式呢？有两种方式：



## 方法一：通过修改配置 ##

Tools -> Options -> Advanced -> General 使用Config Editor编辑配置文件。

mailnews.reply_header_type 一项的值:

    Value                   Elements                                Example
    0                       mailnews.reply_header_originalmessage   -------- Original Message --------
    1 (SeaMonkey default)   authorwrote colon                       Alf Aardvark wrote:
    2 (Thunderbird default) ondate separator authorwrote colon      On 01-01-2007 11:00 AM, Alf Aardvark wrote:
    3                       authorwrote separator ondate colon      Alf Aardvark wrote, on 01-01-2007 11:00 AM:

具体说：

    mailnews.reply_header_authorwrote
        Default: %s wrote
        Thunderbird replaces the %s with the author's name or e-mail address.
    mailnews.reply_header_colon
        Default: : (colon)
        You can add more text here, before the colon or replacing the colon.
    mailnews.reply_header_separator
        Default: , (comma space)
        You can add more text here, before the comma or replacing the comma.
    mailnews.reply_header_ondate
        Default: On %s
        Thunderbird replaces the %s with the date and time.
    mailnews.reply_header_locale
        Default: empty
        Optionally specify a locale for formatting the date and time—for example: en-US
    mailnews.reply_header_originalmessage
        Default: -------- Original Message --------
        Changes the reply header to the text you specify if mailnews.reply_header_type is set to 0.

其中的`mailnews.reply_header_locale`应该可以更改时间日期格式，但实验不成功。

为了使邮件简洁，使用`mailnews.reply_header_type=0`。

## 方法二：通过安装插件 ##
[hange quote and reply format](https://nic-nac-project.org/~kaosmos/changequote-en.html)插件可以构造如下样式的reply header:

    ---- Original message -----
    From: somebody@email_provider.com
    To: you@email_provider.com
    Subject: something
    Date: 09/07/2007


