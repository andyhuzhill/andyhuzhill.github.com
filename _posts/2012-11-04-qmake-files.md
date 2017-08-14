---
layout: post
title: "qmake 项目文件的写法"
description: ""
categories: 
tags: 
- Qt
---
{% include JB/setup %}
* TOC
{:toc}
<hr/>

使用`Qt`编程的都知道，`Qt`中有一套不同于标准`C++`的语法,为了处理这些非标准的东西，就得通过`Qt`提供的一个工具——`qmake`。      
 `qmake`通过读取项目文件来生成一个`Makefile`，然后才能使用标准的`make`工具构建工程。    
 这里来说说怎么写或者是改写这个项目文件。

##实例

 这里有一个项目文件的例子：    

    #-------------------------------------------------
    #
    # Project created by QtCreator 2012-11-04T13:58:26
    #
    #-------------------------------------------------

    QT       += core gui network

    TARGET = Monitor
    TEMPLATE = app

    LIBS += espleak

    SOURCES += main.cpp\
            widget.cpp \
          logindialog.cpp

    HEADERS  += widget.h \
             logindialog.h

    FORMS    += widget.ui \
             logindialog.ui

    TRANSLATIONS += trans.po

    RESOURCES += \
             Resource.qrc


这是一个比较典型的一个项目文件，有定义了使用到的`Qt`的模块，要生成的程序名称，使用的生成程序的模版，要链接的第三方的库，源文件列表，头文件列表，窗体列表，翻译文件，资源文件。

##变量

下面的表格列出了`qmake`能识别的变量，和他们的功能描述：    

<table>
    <tr>
    <td> 变量名</td><td>描述</td></tr>
    <tr>
    <td> CONFIG </td><td> 通用项目配置选项</td></tr>
    <tr>
    <td> DESTDIR </td><td>可执行文件或二进制文件存储的目录</td></tr>
    <tr>
    <td> FORMS </td><td>将要被`uic`处理的窗体文件列表</td></tr>
    <tr>
    <td> HEADERS </td><td> 头文件列表</td></tr>
    <tr>
    <td> QT </td><td> Qt配置选项</td></tr>
    <tr> 
    <td> RESOURCES </td><td> 资源文件列表</td></tr>
    <tr>
    <td> SOURCES </td><td> 源文件列表</td></tr>
    <tr>
    <td> TEMPLATE </td><td>项目使用的模版，这决定了项目编译出来的结果是可执行程序，还是库或插件。</td></tr>
 </table>
 
## PKG-CONFIG

  如果你也接触过gtk+的编程， 你会知道，在编译的时候要用到一个叫做pkg-config的工具来告诉编译器去哪里寻找库文件，在Qt中也可以使用这个工具，使用方法就是在qmake项目文件中加入以下几行：     

    CONFIG += link_pkgconfig
    PKGCONFIG += gtk-2.0

这样， 在qmake生成的Makefile中就会自动加上`pkg-config --libs --cflags gtk-2.0`的语句，就可以使用Qt链接到gtk的库文件了。


## 参考资料
   Qt自己所带的帮助文档已经非常丰富，里面有详细介绍如何去写qmake的项目文件，你只要在帮助的索引处输入qmake，他就会帮你找到。

