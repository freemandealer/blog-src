title: Update kernel image and modules for Cubian
date: 2014-08-25 18:40:39
tags: Note
---

<!--more-->

# Update kernel image and modules for Cubian #

You have a lot of reason to replace the kernel, if you are reading this article.

First, you should setup building environment. FYI, check this if you are going to employ cross compilation.

Compile the kernel and modules using:

	make uImage
	make modules
	make install
	make modules_install
	
However, the later two command won't work well on Cubian/cubieboard. We should do something ourself. That's not very difficult.

First, replace `/boot/uImage` with the new.

Then copy '*.ko' files into `/lib/moduels/<kernel-verion>/kernel`. Make the directory just like it used to be, but with new modules. I did that this way:

	cp -R <directorys' names> /lib/modules/<kernel-version>/kernel/
	find not -name "*.ko" -exec rm {} \;

Reboot. That's it!
	
BACK UP YOUR DATA BEFORE ACTION.