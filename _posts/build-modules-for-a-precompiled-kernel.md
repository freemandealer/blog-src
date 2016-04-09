title: Build Modules For A Precompiled Kernel(Copyright © 2001 Peter Jay Salzman)
date: 2014-11-29 21:40:31
tags: Note
---

Obviously, we strongly suggest you to recompile your kernel, so that you can enable a number of useful debugging features, such as forced module unloading (MODULE_FORCE_UNLOAD): when this option is enabled, you can force the kernel to unload a module even when it believes it is unsafe, via a rmmod -f module command. This option can save you a lot of time and a number of reboots during the development of a module.

<!--more-->

Nevertheless, there is a number of cases in which you may want to load your module into a precompiled running kernel, such as the ones shipped with common Linux distributions, or a kernel you have compiled in the past. In certain circumstances you could require to compile and insert a module into a running kernel which you are not allowed to recompile, or on a machine that you prefer not to reboot. If you can't think of a case that will force you to use modules for a precompiled kernel you might want to skip this and treat the rest of this chapter as a big footnote.

Now, if you just install a kernel source tree, use it to compile your kernel module and you try to insert your module into the kernel, in most cases you would obtain an error as follows:

    insmod: error inserting 'poet_atkm.ko': -1 Invalid module format
    
Less cryptical information are logged to /var/log/messages:

    Jun  4 22:07:54 localhost kernel: poet_atkm: version magic '2.6.5-1.358custom 686 
    REGPARM 4KSTACKS gcc-3.3' should be '2.6.5-1.358 686 REGPARM 4KSTACKS gcc-3.3'
        
In other words, your kernel refuses to accept your module because version strings (more precisely, version magics) do not match. Incidentally, version magics are stored in the module object in the form of a static string, starting with vermagic:. Version data are inserted in your module when it is linked against the init/vermagic.o file. To inspect version magics and other strings stored in a given module, issue the modinfo module.ko command:

    [root@pcsenonsrv 02-HelloWorld]# modinfo hello-4.ko 
    license:        GPL
    author:         Peter Jay Salzman <p@dirac.org>
    description:    A sample driver
    vermagic:       2.6.5-1.358 686 REGPARM 4KSTACKS gcc-3.3
    depends:        
            
To overcome this problem we could resort to the --force-vermagic option, but this solution is potentially unsafe, and unquestionably inacceptable in production modules. Consequently, we want to compile our module in an environment which was identical to the one in which our precompiled kernel was built. How to do this, is the subject of the remainder of this chapter.

First of all, make sure that a kernel source tree is available, having exactly the same version as your current kernel. Then, find the configuration file which was used to compile your precompiled kernel. Usually, this is available in your current /boot directory, under a name like config-2.6.x. You may just want to copy it to your kernel source tree: cp /boot/config-`uname -r` /usr/src/linux-`uname -r`/.config.

Let's focus again on the previous error message: a closer look at the version magic strings suggests that, even with two configuration files which are exactly the same, a slight difference in the version magic could be possible, and it is sufficient to prevent insertion of the module into the kernel. That slight difference, namely the custom string which appears in the module's version magic and not in the kernel's one, is due to a modification with respect to the original, in the makefile that some distribution include. Then, examine your /usr/src/linux/Makefile, and make sure that the specified version information matches exactly the one used for your current kernel. For example, you makefile could start as follows:

    VERSION = 2
    PATCHLEVEL = 6
    SUBLEVEL = 5
    EXTRAVERSION = -1.358custom
    ...
                
In this case, you need to restore the value of symbol EXTRAVERSION to -1.358. We suggest to keep a backup copy of the makefile used to compile your kernel available in /lib/modules/2.6.5-1.358/build. A simple cp /lib/modules/`uname -r`/build/Makefile /usr/src/linux-`uname -r` should suffice. Additionally, if you already started a kernel build with the previous (wrong) Makefile, you should also rerun make, or directly modify symbol UTS_RELEASE in file /usr/src/linux-2.6.x/include/linux/version.h according to contents of file /lib/modules/2.6.x/build/include/linux/version.h, or overwrite the latter with the first.

Now, please run make to update configuration and version headers and objects:

    [root@pcsenonsrv linux-2.6.x]# make
    CHK     include/linux/version.h
    UPD     include/linux/version.h
    SYMLINK include/asm -> include/asm-i386
    SPLIT   include/linux/autoconf.h -> include/config/*
    HOSTCC  scripts/basic/fixdep
    HOSTCC  scripts/basic/split-include
    HOSTCC  scripts/basic/docproc
    HOSTCC  scripts/conmakehash
    HOSTCC  scripts/kallsyms
    CC      scripts/empty.o
    ...
                    
If you do not desire to actually compile the kernel, you can interrupt the build process (CTRL-C) just after the SPLIT line, because at that time, the files you need will be are ready. Now you can turn back to the directory of your module and compile it: It will be built exactly according your current kernel settings, and it will load into it without any errors.
