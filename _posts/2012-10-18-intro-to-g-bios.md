---
layout: post
title: "g-bios 一个中国人写的bootloader"
description: ""
categories: 
- ARM
tags: 
- bootloader
- g-bios
---
{% include JB/setup %}




## 概况
 g-bios 计划的目的是为Android和通用嵌入式Linux系统设计和开发一个通用体系bootloader/BIOS。它是由 Intel、IBM、Qualcomm、AMD 的几名资深软件工程师与开源社区共同研发的一个 Bootloader。    
 项目主要负责人是有着业界狂人之称[2]的Conke Hu(毕业于浙江大学，原Intel、AMD公司资深软件工程师/项目经理，AMD Chipset Linux Kernel负责人，Linux Kernel、g-bios等开源项目开发者)。目前支持的芯片有S3C2410、S3C2440、at91sam9263、at91sam9261。    
<s>现在这个项目似乎处于停滞状态，代码最新的提交是在2009年8月24日。在ChinaUnix网站上发现这个项目一直有活动，不知道为什么，googlecode上的代码提交只到2009年? </s>    
OK，我在github上面发现了他的代码，看来，作者也是把他的代码放到github上面了，我就有使用github，居然木有发现！最新的代码地址：

 [maxwit/g-bios](https://github.com/maxwit/g-bios)



## 特点

-  自动检测有待烧录的image文件类型，并智能自动烧录。
-   支持多种文件系统，包括YAFFS2、JFFS2、CRAMFS、NFS等。
-  支持两种用户界面：GUI(类似传统PC BIOS)和命令行模式(面向嵌入式系统)。
-  命令行自动补全(Tab键)及历史记录(上、下键)支持。
-  Flash(MTD)分区支持，帮助Linux、Android内核识别分区。
-  自动设置Linux内核启动参数(Linux kernel command line)，极大地降低了参数设置的复杂度并减少了启动出错的概率。当然，同时也支持手动设置，以满足特殊要求。
- 常用命令具有记忆功能。如boot命令，它能记住用户输入的参数，以后只需简单输入boot即可。
-  引入全新的架构及NB(Never Burn Down，烧不死)技术。开发人员可在没有仿真器的情况下轻松完成整个g-bios的开发。
- 支持 kermit。

## `g-bios` 框架图

![g-bios 框架图](/images/g-bios.jpg) 


## 参考资料

[项目主页地址](https://code.google.com/p/maxwit/)

