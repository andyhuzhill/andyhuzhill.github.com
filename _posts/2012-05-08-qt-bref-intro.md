---
comments: true
date: 2012-05-08 14:52:10
layout: post
title: Qt 入门系列 （一） - Qt 简介
categories:
- Linux
tags:
- C++
- Qt
- GUI
---

{% include JB/setup %}
* TOC
{:toc}
<div style="border-bottom: 1px solid #ccc;line-height: 1.3em;"></div>
# Qt – 一个跨平台应用程序和UI开发框架


Qt 是一个跨平台应用程序和 UI 开发框架。使用 Qt 您只需一次性开发应用程序，无须重新编写源代码，便可跨不同桌面和嵌入式操作系统部署这些应用程序。



### 功能






  1. 直观的 C++ 类库


  2. 跨桌面和嵌入式操作系统的移植性


  3. 具有跨平台 IDE 的集成开发工具


  4. 在嵌入式系统上的高运行时间性能，占用资源少










### Qt 支持以下平台:
	
  * 嵌入式Linux

	
  * 苹果 Mac OS X

	
  * 微软 Windows

	
  * 桌面Linux

        
  * 微软Windows CE/Mobile

        
  * Symbian

        
  * Meego





### 优势






  * #### 面向对象


　　Qt 的良好封装机制使得 Qt 的模块化程度非常高，可重用性较好，对于用户开发来说是非常 方便的。 Qt 提供了一种称为 signals/slots 的安全类型来替代 callback，这使得各个元件之间的协同工作变得十分简单。


  * #### 丰富的 API


　　Qt包括多达 250 个以上的 C++ 类，还提供基于模板的 collections， serialization， file， I/O device， directory management， date/time 类。甚至还包括正则表达式的处理 功能。

　
  * #### 支持 2D/3D 图形渲染，支持 OpenGL


　
  * #### 大量的开发文档


　
  * #### XML 支持


　
  * #### Webkit 引擎的集成，可以实现本地界面与Web内容的无缝集成


　但是真正使得 Qt 在自由软件界的众多 Widgets (如 Lesstif，Gtk，EZWGL，Xforms，fltk 等等)中脱颖而出的还是基于 Qt 的重量级软件 KDE 。




####   使用Qt开发的著名软件 




    1. Autodesk MotionBuilder, professional 3D character animation software


    2. Autodesk Maya, 3D建模和动画软件


    3. EAGLE, tool for designing printed circuit boards（PCBs） 是一款低价格、界面丰富、人性化、易于学习和使用且功能强大的原理图和PCB设计工具，它有很多高级功能：例如在线正反向标注功能、批处理命令执行脚本文件、覆铜以及交互跟随布线器等功能。


    4. Google地球（Google Earth）：三维虚拟地图软件。


    5. QCad：一个用于二维设计及绘图的CAD软件


    6. Opera：著名的网页浏览器。


    7. Skype：一个使用人数众多的基于P2P的VOIP聊天软件。


    8. SMPlayer：跨平台多媒体播放器


    9. VirtualBox：虚拟机软件。


    10. 极品飞车


    11. 豆瓣播放器


    12. 金山 WPS : 中国最著名的民族软件最近也用Qt改写了，并且正在开发Linux和Mac版本


    13. ……………




参考资料：
 [Qt官方网站](http://qt.nokia.com)



