---
comments: true
date: 2012-05-21 13:13:31
layout: post
title: 用单片机做一个简单的计算器
categories:
- Mylife
- ProgramminginC
- ProgrammingonMCU
tags:
- 80c51
- C
- MCU
- 电子小制作
---

{% include JB/setup %}
[![](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/05/calc.jpg-300x205.png)](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/05/calc.jpg.png)
这几天，学院举办的那个电子设计比赛，我们学院的同学太不给力啦，才出了四个作品。被科技部的同学找到说要我们无论如何要把之前报的作品作出来，最好还能多做几个作品，于是我花了两天时间做了一个简单的计算器，实现了一些简单的加减乘除，本来还想做正弦函数，余弦函数的计算的，但是有点麻烦，就木有弄了。
首先，是画电路图，原理还是挺简单的，就是单片机接上键盘，接上显示器，然后读取键盘，显示计算结果就是了。原理图在上面已经有了，[这里](http://andylinux.sinaapp.com/html/ylt.pdf)也有一份PDF格式的原理图大图。画图软件是Linux系统下的KiCAD，感觉使用起来还是挺好用的。
电路板是完全用洞洞板连接起来的，不太想去学院的制板机那做板子了，之前做了好几块板子都作废了，有点受伤。
[![电路板背面，](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/05/图像0133-300x225.jpg)](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/05/图像0133.jpg)焊接焊得还是挺难看的：P
[![通电后的效果图](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/05/图像01352-300x225.jpg)](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/05/图像01352.jpg)
单片机是一片AT89S52，8051内核的单片机。
显示器用的是1602液晶，一开始写的简单程序，显示的结果都不太符合自己想要的效果，然后稍稍仔细的读了一下1602的数据手册，把它的各个指令都试了试，基本上能做出想要的结果了。
这里是一个演示视频

<embed src="http://player.youku.com/player.php/sid/XNDAwMTM1MzY4/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>


源代码全部都在[这里](https://github.com/andyhuzhill/passlocker)了，你可以下载下来看看,也可以添加一些功能哦。
编译器使用的是SDCC 3.1.4,使用Eclipse管理和编写代码。

