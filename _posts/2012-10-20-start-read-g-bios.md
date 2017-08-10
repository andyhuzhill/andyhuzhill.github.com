---
layout: post
title: "开始阅读g-bios源代码"
description: ""
categories: 
-   ARM
tags:
-   bootloader
-   g-bios
---

* TOC
{:toc}
<hr/>


# g-bios 阅读笔记


 **最新的`g-bios`的代码在`github`网站上，所以下面的内容有些已经`Out-of-date`了，不过你还是可以照着做做看的。**
 **最新的`g-bios`代码可以使用以下命令获得：**
 
    git clone git://github.com/maxwit/g-bios.git


 关于`g-bios`的介绍，我已经在[上一篇文章]({{baseurl}}{{page.previous.url}})中介绍过了。现在，我要开始阅读`g-bios`的代码。
 
## 获取`g-bios`代码
首先要获取代码，要安装svn，安装好后在命令行输入以下代码：

    svn co http://maxwit.googlecode.com/svn/trunk/ maxwit-read-only


下载下来的代码中，`g-bios`的源代码在 `maxwit-read-only/g-bios` 目录下。

## 目录结构
    
├── app   
│   ├── boot    
│   ├── device_mamger    
│   ├── flash    
│   ├── memory    
│   ├── net    
│   ├── serial    
│   ├── shell   
│   └── sys    
├── arch    
│   └── arm    
│       ├── atmel    
│       └── samsung    
├── boot   
│   └── arm   
│       ├── atmel    
│       └── samsung    
├── device    
│   ├── audio    
│   ├── bus   
│   ├── flash   
│   ├── graphics    
│   ├── interrupt    
│   ├── key    
│   ├── led    
│   ├── net    
│   ├── rtc    
│   ├── timer    
│   ├── touchscreen    
│   ├── uart    
│   └── watchdog    
├── doc    
│   ├── Chinese    
│   └── English    
├── filesys    
│   ├── base    
│   ├── ext4    
│   ├── fat32    
│   ├── generic    
│   ├── gfs    
│   ├── ntfs    
│   ├── yaffs    
│   └── yaffs2    
├── include     
│   ├── arch    
│   ├── core    
│   ├── flash     
│   └── net    
├── lib    
│   ├── arm    
│   └── stdlib     
├── mm     
│   ├── arm     
│   └── heap    
└── thread     
           
`app`目录下是一些`g-bios`可用的命令的源代码。`arch`目录下是`g-bios`下半部分。
`boot`目录下是`g-bios`上半部分。`device`目录下是一些硬件驱动(大多为空的)。`doc`目录下是文档。
`filesys`目录下是文件系统驱动(大多为空的)。`include`目录下是`g-bios`包含的一些头文件。
`lib`目录下是一些C语言运行库。`mm`下是内存管理的函数。

## 开始

我们按照`g-bios`的启动顺序开始阅读。`g-bios`设计上分为两部分，上半部分和下半部分，上半部分短小精悍，完成CPU和Memory的初始化，并顺利引导下半部分。     
上半部分设计简单，调试周期短，完成后就固化在特定的引导区中不再更改；开发人员可在没有仿真器的情况下大胆开发下半部分代码（即g-bios主体），事实上，只需一根串口数据线应能轻松完成整个g-bios的开发。启动代码的地址无关性带来的麻烦？没有了！因为bug或不小心改错了代码，甚至是数据线接触不良而导致启动黑屏？永远不可能出现了！     
在调试完成并正试发布的产品时，若有必要，也可将上下两部分可合成一个整体——只需一个命令重新编译即可。

### head.S

上半部分的第一段运行的代码在`boot/arm/head.S`中。

    #include <g-bios.h>
    #include <arch/arm.h>
    #include <core/kermit.h>
    #include <autoconf.h>

首先是包含各种头文件。 
* `g-bios.h`几乎是所有`g-bios`的程序中要包含的头文件，包含定义的类型、符号等等。     
* `arch/arm.h`中定义了`ARM`芯片几种运行模式的宏，还有中断处理的宏函数。     
* `core/kermit.h`中定义了`kermit`文件传输协议的一些宏。     
* `autoconf.h`中定义了`CPU`型号、`Uart`、`flash`、还有网络设置、`NFS`地址定义、串口名定义。*这个文件是由`configure`程序生成的。*     

接下来是定义两个符号 `GTopHalfEntry`(上半部分的入口），`Hang`（一个死循环的地址）。    

然后是中断向量设置。不管是什么中断，都跳到复位向量处执行：        

    msr cpsr, #(ARM_MODE_SVC | ARM_INIT_MASK) @ 管理模式，关闭 IRQ和FIQ
    
    mov sp, #INIT_STACK_BASE                  @ INIT_STACK_BASE 定义在 include/arch/目录下的与芯片对应的头文件中
    
    bl main                                   @跳转到 boot/init.c文件中的main函数 

下面注释掉的汇编代码，现在已经使用C语言改写了(文件在`boot/init.c`)


### init.c

 `init.c`的中就是实际的初始化代码了。`main`函数的内容很简单，如下：

{% highlight c %}
int main(void)
{
	const char *pchBanner = "\n\n" // CLRSCREEN
		"\t+----------------------------------+\n\r"
		"\t|     Welcome to MaxWit g-bios!    |\n\r"
		"\t|  (http://maxwit.googlecode.com)  |\n\r"
		"\t|        ["   BUILD_TIME  "]       |\n\r"
		"\t+----------------------------------+\n\r";
	GtInitSoC();                     //初始化芯片
	GtInitUart();                    //初始化Uart
	puts(pchBanner);                 //打印上面的欢迎信息
	ReadID();                        //读取芯片信息
	GtInitMemCtrl();                 //初始化内存控制
	GtInitNand();                    //初始化flash
	BootMenu();                      //显示启动菜单
	return -1;
}
{% endhighlight %} 

在目前的代码中，这几个函数在不同平台使用了不同的编程语言实现。    

`at91sam926x`是在汇编语言程序`boot/arm/atmel/at91sam926x.S`中     
实现了`GtInitSoc`、`GtInitUart`、`GtInitMemCtrl`、`GtInitNand`,   
`At91NandIsReady`,`GtUartReadByte`,`GtUarteWriteByte`。         

`s3c24xx`是在`boot/arm/samsung/s3c24xx.c`中实现了`GtInitSoc`、     
`GtInitMemCtl`. `boot/arm/samsung/uart_s3c24xx.c`中实现了`GtInitUart`,    
`GtUartReadByte`,`GtUartWriteByte`,`InitGpio`,`S3c24xxUartInitOne`.     
`boot/arm/samsung/nand_s3c24x0.c`中实现了`S3c24x0NandIsReady`,`GtInitNand`。

`GtInitSoc` 初始化芯片的时钟。





