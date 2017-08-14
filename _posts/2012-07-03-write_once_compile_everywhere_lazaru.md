---
comments: true
date: 2012-07-03 01:26:08
layout: post
title: 一次编写、到处编译——Lazarus多平台编译设置
categories:
- Linux
- pascal
tags:
- GCC
- lazarus
- Pascal
- 跨平台
---

{% include JB/setup %}



总所周知，Java的一个特点就是“一次编写，处处运行”，不过有一个缺点就是：你得在你要运行的机器上先运行一个jre（java runtime environment，java运行环境），而且Java程序的执行效率不如本地原生代码。为了实现跨平台运行，又有人提出了新的想法，“一次编写，到处编译”，比较著名的就是Qt这个图形界面框架，使用编程语言是C++。Qt的一个缺点就是依赖的库文件比较大，尤其是在嵌入式系统上，有时候可能希望能使用更少的存储空间实现同样的图形界面。而且，当你使用新版本的Qt编写程序时，又会依赖更高版本的动态连接库。今天我要介绍的是另一个编程工具——Lazarus，可能大家都没听过，它使用的编程语言是Free Pascal，一种面向对象的高级Pascal语言。它也是一个开源软件，而且被移植到了各个平台和系统上，比如Windows、Linux、Mac OS，i386平台，arm平台、x86_64平台......




### 安装


lazarus的官方网站是[http://www.lazarus.freepascal.org/](http://www.lazarus.freepascal.org/),你可以点击左边的download链接进入下载页面，选择你想要下载的版本。我使用的Linux系统，下面我也是以Linux系统来介绍如何配置lazarus的跨平台编译。一般的Linux系统的软件包管理系统里面都已经包含了lazarus，可以使用如下命令安装：

    
    
    #ubuntu 使用如下命令
    sudo apt-get install lazarus
    #arch linux 使用如下命令
    sudo pacman -S lazarus
    


你也可以下载Lazarus的源码包，自己动手编译安装
    
    
    wget http://downloads.sourceforge.net/project/lazarus/Lazarus%20Zip%20_%20GZip/Lazarus%200.9.30.4/lazarus-0.9.30.4-src.tar.bz2?r=http%3A%2F%2Fsourceforge.net%2Fprojects%2Flazarus%2Ffiles%2FLazarus%2520Zip%2520_%2520GZip%2FLazarus%25200.9.30.4%2F&ts;=1341246028&use;_mirror=ncu
    tar xvf lazarus-0.9.30.4.src.tar.gz
    cd lazarus-0.9.30.4
    make clean install
    


Windows用户下载安装程序包就和普通windows程序一样安装。




### 初识Lazarus


[![lazarus界面](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/07/2012-07-03-002311_1364x733_scrot-300x161.png)](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/07/2012-07-03-002311_1364x733_scrot.png)
如果你用过delphi，那么你一定会对lazarus非常熟悉，同样是使用pascal作为编程语言，同样是RAD（Rapid Application Development）工具，lazarus已经在一些方面比delphi还要强大了。





### 添加windows编译支持



我现在使用的是linux系统，我们都知道linux系统和windows系统使用的可执行文件的格式是不一样的，Windows里一般是PE文件，而Linux一般是ELF文件。那么Linux下就不能写Windows的程序了吗？非也，非也。我们首先要感谢GNU计划提供了GCC这个强大的开源编译器，是我们也能够在Linux下编写Windows的程序。我现在就要讲安装linux下的windows程序交叉编译器

###### （什么叫做交叉编译器？就是编译生成的代码不是在本机环境里运行的，而是在其他环境里运行的编译器。如果你进行过嵌入式开发，会发现你使用的编译器都是交叉编译器。）


mingw32-gcc,这个编译也有windows的版本，我现在要用的就是linux的版本。先从它的官方网站上下载它的源代码，然后编译它。这个比较麻烦，我就直接使用已经编译好的版本，使用如下命令安装：

    
    
    # ubuntu
    sudo apt-get install gcc-mingw32
    # Arch Linux
    sudo pacman -S mingw32-gcc
    


我发现这个程序在不同发行版中的包的名称也不一样，而且安装后，可执行文件的名称也不一样。ubuntu（11.04）里面的名称是i586-mingw32msvc-gcc，而ArchLinux（3.4.2-2）里面的名称是i486-mingw32-gcc。这点需要大家注意一下。


### 编译fpc和lazarus


 安装好交叉编译器之后，我们就要重新编译一下lazarus，使之支持新编译器。首先交叉编译fpc。

    
    
    cd /usr/lib/fpc/src/
    fpcmake crossinstall OS_TARGET=win32 CPU_TARGET=i386 INSTALL_PREFIX=/usr
    


编程安装之后，fpc的RTL就编程成windows版本了。


然后启动lazarus,如果你是使用命令安装软件源里早已编译好的lazarus，这里需要管理员身份启动Lazarus，这是因为lazarus的默认安装路径（/usr/lib/lazarus)普通用户没有写的权限。如果你是自己从源代码安装，则进入你安装的位置启动即可。

###### （lazarus可以配置成中文界面，不过我还是以默认的英文界面来讲解吧）

###### 
然后依次点击如下菜单  Tools-Configure "building lazarus"-Advaned Build Options。
选择 Target OS 为 Win32 Target CPU 为 i386。在左边的单选列表中，将LCL和Package registration前面的按钮设置成中间的按钮
其他的设置成第一个按钮（语言可能表述不清，我第一次也没看懂，现在放一张图片应该一看就知道了）
[![](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/07/2012-07-03-005347_706x556_scrot.png)](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/07/2012-07-03-005347_706x556_scrot.png)
然后点击build按钮等待完成，就可以了。



### 设置项目


 现在我们已经可以使用我们的lazarus在linux下写出windows的程序了，我们需要在IDE中对项目在进行一下设置：
依次点击  Project / Compiler Options / Code generation / Target OS (-T): Win32
再设置 Project / Compiler Options / Code generation / Target CPU family (-P): i386
如图所示
[![](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/07/2012-07-03-010128_706x526_scrot.png)](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/07/2012-07-03-010128_706x526_scrot.png)
设置Project / Compiler Options /Paths/LCL widget type ：Win32/Win64
[![](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/07/2012-07-03-011416_703x527_scrot.png)](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/07/2012-07-03-011416_703x527_scrot.png)

然后就是直接编译就能生成需要的文件了。




### 其他平台支持


上面讲的是i386平台的linux上编译i386平台上的windows程序，同样的，我们只要有了交叉编译器，就可以像上面一样的方法生成不同的跨平台的文件。


Linux to Wince  



linux上有一个wince的编译器，叫做arm-wince-cegcc-gcc，在archlinux下安装命令如下

    
    
    sudo yaourt -S arm-wince-cegcc-gcc
    


安装到的目录是/opt/cegcc/bin，将这个路径添加到.bashrc 环境变量中去 

    
    
    echo "export PATH=$PATH:/opt/cegcc/bin" >> ~/.bashrc
    


然后重新编译fpc的源代码

    
    
    sudo make OS_TARGET=wince CPU_TARGET=arm INSTALL_PREFIX=/usr
    


重新编译lazarus


然后依次点击如下菜单  Tools-Configure "building lazarus"-Advaned Build Options。
选择 Target OS 为 WinCE Target CPU 为 arm。在左边的单选列表中，将LCL和Package registration前面的按钮设置成中间的按钮
其他的设置成第一按钮。
点击build按钮。



项目设置

  

依次点击  Project / Compiler Options / Code generation / Target OS (-T): wince
再设置 Project / Compiler Options / Code generation / Target CPU family (-P): arm
设置Project / Compiler Options /Paths/LCL widget type ：wince(beta)


Linux到 ARM-linux   



首先安装交叉编译器，网上有很多arm的linux编译器，可以随便下载一个安装。
编译fpc源代码

    
    
    sudo make OS_TARGET=linux CPU_TARGET=arm INSTALL_PREFIX=/usr
    


重新编译lazarus


然后依次点击如下菜单  Tools-Configure "building lazarus"-Advaned Build Options。
选择 Target OS 为 linux Target CPU 为 arm。在左边的单选列表中，将LCL和Package registration前面的按钮设置成中间的按钮
其他的设置成第一按钮。
点击build按钮。



项目设置

  

依次点击  Project / Compiler Options / Code generation / Target OS (-T): Linux
再设置 Project / Compiler Options / Code generation / Target CPU family (-P): arm
设置Project / Compiler Options /Paths/LCL widget type ：gtk2
不过 如果是嵌入式linux的话，LCL widget type 选择gtk2,说明嵌入式linux上要运行X11,所以编译的时候要把嵌入式的X11运行库编译进程序里面。



### windows下开发wince程序


 在windows下开发wince程序，只需要安装win32的lazarus和lazarus-cross-arm-wince-win32.exe就可以啦，生成wince程序只要像上面的设置一下项目就行了。



### 总结


通过上面的说明，我们知道了，只要有gcc移植上去，而且可以交叉编译它的运行库，你就可以在自己喜欢的系统和平台上编写其他平台和环境的程序了。以上都是使用lazarus写程序，其实也可以直接用交叉编译的gcc写C语言的程序，只是如果是使用lazarus写图形界面程序的话，跟本不用修改代码，重新编译就可以啦。






#### 参考资料


 [Setup Cross Compile For ARM](http://wiki.freepascal.org/Setup_Cross_Compile_For_ARM#Make_FPC_able_to_cross_compile_for_arm-linux)
