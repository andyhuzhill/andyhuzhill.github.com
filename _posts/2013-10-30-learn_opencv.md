---
layout: post
title: "学习OpenCV"
description: "OpenCV 简介"
category: OpenCV 
tags:
- OpenCV
- 机器视觉
- 图像处理 
---
{% include JB/setup %}


最近上《视频信号处理》的课，兼之对于图像处理也挺感兴趣的，于是去网上下载了OpenCV这个著名的开源图像处理库。

## OpenCV简介
OpenCV是Intel®开源计算机视觉库。它由一系列C函数和C++类构成，实现了图像处理和计算机视觉方面的很多通用算法。

目前(2013-10-30)的最新版本是2.4.6，早期的OpenCV是用C语言写的，所以你可以在网上找到各种源代码或者是书籍，里面都是讲的C语言接口的程序。而OpenCV2.0开始全面改用CPP写了，虽然也提供了C语言的接口，但是推荐使用C++的方式使用。当然，它也提供了其他语言的一些接口，比如python,比如java，现在OpenCV就有Android和iOS上面的官方移植。

## OpenCV安装

在不同平台下OpenCV有不同的安装方法，OpenCV的源代码是托管在sourceforge上面的，[下载地址](http://opencv.org/downloads.html)

OpenCV for Windows 下载下来是一个exe文件，直接执行安装就可以了。

### 在linux下安装，

你既可以下载源代码编译安装，也可以使用发行版的软件源里自带的OpenCV。

我是Debian 7.0的用户，她的源里面有2.3.1版的OpenCV。 下面就讲一下我的安装过程吧。

打开终端，在终端中输入以下命令

    $ sudo apt-get install libopencv-dev opencv-doc python-opencv

上面的命令的意思是 下载OpenCV的开发库和文档
最后一个python-opencv是可选的，如果你要通过Python来使用OpenCV的话，可以装上这个包。

安装完成后在目录(file:///usr/share/doc/opencv-doc/opencv_tutorials.pdf.gz)
下是OpenCV的一个与你安装的版本相配套的英文教程。这个是最新最权威的帮助信息，包含了OpenCV各个模块和功能的说明，每一个学习OpenCV的同学都应该去看看。


## OpenCV的使用

在上面提到的那个教程里面有详细介绍了OpenCV在各个平台下的安装和使用，我这里只简单介绍一下在linux下的使用方法。

### 编写代码

下面我们用一个简单的读取图片，并显示的程序作为开始。

使用任何你喜欢的文本编辑器，输入以下代码：

{% highlight cpp %}

#include <iostream>

 // 以前的版本里面 可能看到 #inlcude <cv.h>
 // 那是C语言接口的头文件，现在CPP版本推荐如下写法
#include <opencv2/core/core.hpp>     
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/imgproc/imgproc.hpp>

using namespace std;  // 为了使用cerr cout 等等
using namespace cv;   // 为了直接使用opencv的函数 而不用加一个cv::


// 读取命令行第二个参数作为图片的文件名 并显示出来
int main(int argc, char *argv[])
{
    if(argc < 2) {  //命令行参数个数不对
        cerr << "Usage: " << argv[0] << " imagefiletoload" << endl;
        return -1;
    }
    
    Mat img = imread(argv[1], CV_LOAD_IMAGE_COLOR);  //读入图像文件
    
    //使用highgui里面的函数创建一个窗口
    namedWindow("Pictures", CV_WINDOW_AUTOSIZE);
    
    //在之前创建的窗口内显示读入的图像
    imshow("Pictures", img);

    //等待按键按下，否则程序会一闪而过
    waitKey(0);

    return 0;
}   

{% endhighlight c %}


### 编译
做过C/C++语言开发的应该都知道，在编译和链接的过程中，需要开发者告诉编译器去哪里寻找你使用的库的头文件和库文件。

Linux下的程序一般将头文件放在/usr/include 目录下，将库文件放在/usr/lib 目录下。

#### 命令行编译
使用GCC编译器的语法就是下面这一句话

    $ gcc -o test test.c -I/usr/include/opencv  -lopencv_core -lopencv_imgproc -lopencv_highgui
    -lopencv_ml -lopencv_video -lopencv_features2d -lopencv_calib3d
    -lopencv_objdetect -lopencv_contrib -lopencv_legacy -lopencv_flann 
    
这么长一句话，在每次编译使用了很多库的程序时都敲一遍是不可能的，你可很难记住每一个库的头文件地址和库文件名，于是就有了很多的帮助我们建立工程的工具。

首先介绍一下pkg-config， pkg-config
    是一个提供从源代码中编译软件时查询已安装的库时使用的统一接口的计算机软件,它主要提供了以下几个功能：

1.检查库的版本号。如果所需要的库的版本不满足要求，它会打印出错误信息，避免链接错误版本的库文件。

2.获得编译预处理参数，如宏定义，头文件的位置。

3.获得链接参数，如库及依赖的其它库的位置，文件名及其它一些连接参数。

4.自动加入所依赖的其它库的设置。

在使用前，我们说说 pkg-config 的原理，pkg-config并非精灵，可以凭空得到以上信息。事实上，为了让pkg-config可以得到这些信息，要求库的提供者，提供一个.pc文件。比如gtk+-2.0的pc文件内容如下：

    prefix=/usr
    exec_prefix=/usr
    libdir=/usr/lib
    includedir=/usr/include
    target=x11

    gtk_binary_version=2.4.0
    gtk_host=i386-redhat-linux-gnu

    Name: GTK+
    Description: GIMP Tool Kit (${target} target)
    Version: 2.6.7
    Requires: gdk-${target}-2.0 atk
    Libs: -L${libdir} -lgtk-${target}-2.0
    Cflags: -I${includedir}/gtk-2.0
    
 这个文件一般放在 /usr/lib/pkgconfig/ 或者 /usr/local/lib/pkgconfig/
 里，当然也可以放在其它任何地方，如像 X11 相关的pc文件是放在 /usr/X11R6/lib/pkgconfig下的。为了让pkgconfig可以找到你的pc文件，你要把pc文件所在的路径，设置在环境变量 PKG_CONFIG_PATH 里。

 使用 pkg-config 的 –cflags 参数可以给出在编译时所需要的选项，而 –libs
 参数可以给出连接时的选项。例如，假设一个 sample.c 的程序用到了 Glib
 库，就可以这样编译：
 
    $ gcc -o sample `pkg-config –-cflags --libs glib-2.0` sample.c
    
可以看到：由于使用了 pkg-config工具来获得库的选项，所以不论库安装在什么目录下，都可以使用相同的编译和连接命令，带来了编译和连接界面的统一。

使用 pkg-config 工具提取库的编译和连接参数有两个基本的前提：

*.库本身在安装的时候必须提供一个相应的 .pc 文件(不这样做的库说明不支持 pkg-config工具的使用)。

*.pkg-config 必须知道要到哪里去寻找此 .pc 文件。

我们现在要使用pkg-config来找到OpenCV的库，只需要使用以下的命令

    $ gcc -o test test.c `pkg-config --cflags --libs opencv`
    
注意上面有一个''`''号 是反撇号，位于键盘的Tab键上面。


#### 使用Makefile

当然上面的方法也还是有点麻烦，需要在命令行上敲这么长一串，有些人可能使用过make这个工具，于是，我们可以再次将上面的方法改成使用Makefile来做。

下面是一个示例的Makefile

     PROJECT:=test
     CXX:= g++
     CC := gcc
     CXXFLAGS := `pkg-config --cflags --libs opencv`
     OBJECTS:= test.cpp
  
     all:
        $(CXX) -o $(PROJECT) $(CXXFLAGS) $(OBJECTS)
        
     clean:
        rm -rf *.o $(PROJECT)

需要注意的是，Makefile有他特殊的语法，在目标的下一行，命令开始的前面必须有一个Tab。

写好这个Makefile之后，在Makefile所在目录下输入make命令，make就会自动调用编译器编
译生成可执行程序了。具体的Makefile的语法请自己去找相关资料。

#### 使用Cmake

如果你自己亲手编译过OpenCV的源代码，会发现他是使用cmake这个工具管理的。OpenCV的教程里面也推荐使用cmake来编译使用了OpenCV的程序。cmake其实可以看作一个能够自动生成Makefile的工具，上面的Makefile的写法，是不是觉得Makefile的写法也有点麻烦？

那么就使用cmake吧，他的语法比Makefile较简单，而且没有太多格式要求。下面是一个编译简单的使用了OpenCV的程序的cmake配置文件，CMakeLists.txt

    # 设置工程名（可选）
    PROJECT(main)
    # 找到OpenCV的库 这个选项类似pkg-config的功能
    find_package(OpenCV REQUIRED)
    # 下一行是cmake的一个要求，申明需要的cmake版本
    CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
    # 下一行也是可选的 添加一个源文件目录变量 变量名为DIR_SRCS 变量的值为 "."
    # (即当前目录）
    AUX_SOURCE_DIRECTORY(. DIR_SRCS)
    # 添加可执行程序
    ADD_EXECUTABLE(main ${DIR_SRCS})
    # 将OpenCV的库链接到可执行程序上
    target_link_libraries(main ${OpenCV_LIBS})
    
写好这个文件之后，在当前目录下执行 cmake .

    andy@debian:~/Myprogram/opencv/cmake$ cmake .
    -- The C compiler identification is GNU 4.7.2
    -- The CXX compiler identification is GNU 4.7.2
    -- Check for working C compiler: /usr/bin/gcc
    -- Check for working C compiler: /usr/bin/gcc -- works
    -- Detecting C compiler ABI info
    -- Detecting C compiler ABI info - done
    -- Check for working CXX compiler: /usr/bin/c++
    -- Check for working CXX compiler: /usr/bin/c++ -- works
    -- Detecting CXX compiler ABI info
    -- Detecting CXX compiler ABI info - done
    -- Configuring done
    -- Generating done
    -- Build files have been written to: /home/andy/Myprogram/opencv/cmake
    andy@debian:~/Myprogram/opencv/cmake$ 

屏幕上走过一大串字符后，就会在当前目录生成一个Makefile，这个时候直接执行make命令。

    andy@debian:~/Myprogram/opencv/cmake$ make
    Scanning dependencies of target main
    [100%] Building CXX object CMakeFiles/main.dir/main.cpp.o
    Linking CXX executable main
    [100%] Built target main
    andy@debian:~/Myprogram/opencv/cmake$ 

程序就编译好了！ 

详细的Cmake的用法，请自己去搜索相关资料。

以上讲了好几个编译程序的方法，你有没有被吓到，说编译OpenCV的程序这么麻烦啊，其实只要
掌握一种你觉得方便的就可以了。上面讲的都没有用到IDE（集成开发环境），对于习惯了使用
Windows的同学来说，可能觉得没有IDE不习惯吧，那我下面重点讲一下使用QtCreator来进行
OpenCV的开发。

### 使用Qt结合OpenCV 

之所以使用Qt，是因为我之前写图形界面程序都是用的Qt，而Qt Creator这个IDE也是很不错
的，网上有很多地方都讲了怎么在Visual Studio底下开发OpenCV程序，我就不在这里讲了。
Qt也是一个C++的库，是一个很流行的开源图形界面库。

首先讲安装Qt，Qt的最新版是5.1.1（2013-10-31）安装程序可以在他的官网上下载到
[下载地址](http://qt-project.org/downloads)

你可以自由选择适合你的操作系统的版本，
目前官网上下载的程序貌似都包含了Qt的二进制库和IDE（Qt Creator）。

针对Windows版的有好几个，我这里要说明一下。
[Qt 5.1.1 for Windows 32-bit (MinGW 4.8, OpenGL, 666 MB)(Info)](http://download.qt-project.org/official_releases/qt/5.1/5.1.1/qt-windows-opensource-5.1.1-mingw48_opengl-x86-offline.exe)
这一个版本的Qt是使用GCC（MinGW）编译的库，

下面的带有 VS 字样的都是使用Visual Studio
也就是微软的编译器编译的库。如果你想要在Visual Studio下开发Qt程序，可能还要安装
下面 Other downloads 里面的 Visual Studio Add-in 插件

ubuntu或者debian下面还可以安装软件源里面自带的Qt sdk 使用如下命令即可

    $ sudo apt-get install qt-sdk

这样就会将所有开发Qt程序所需的软件安装好了。

下面，我会图文并茂的讲如何使用Qt Creator建立一个OpenCV的工程，你的Qt
Creator的新建工程界面可能与我的稍有不同，但是只是有一点点不同而已。

点击文件-新建文件或工程，找到 “纯C++语言项目”
（或者是纯C语言项目，如果你使用C语言）
![](/images/opencv/QtCreatorNew1.png)

在后一个页面写入工程名，注意：如果你是在windows下 工程名不要使用中文。
![](/images/opencv/QtCreatorNew2.png)


然后的几个是配置是否使用代码版本控制工具，选默认的然后下一步到完成就可以了。

![](/images/opencv/QtCreatorNew3.png)

然后就要在工程中添加OpenCV的库，这里在不同的环境下的配置方式是不一样的，我下面所说的
是Debian 7.0 下 使用 sudo apt-get install libopencv-dev 后的配置方法，你会发
现真的很简单。

在左边工程管理器的工程名上点击右键，出现下图 然后选择添加库

![](/images/opencv/QtCreatorAddOpenCV1.png)

选择最下面那个系统包，我们可以看到，Qt其实也是利用了pkg-config的功能实现OpenCV库的链接。

![](/images/opencv/QtCreatorAddOpenCV2.png)

下一步 输入包的名称为opencv 点击下一步 直到完成即可。

然后我们可以看到工程文件（未命名3.pro)里面多了两行

    unix: CONFIG += link_pkgconfig
    unix: PKGCONFIG += opencv
 
 前面的unix代表在类unix环境下添加以下配置。
 
 这样设置，编译的时候就会连接到OpenCV的库了。
 
 好吧，有些人可能会问，如果我不是使用软件源里面的OpenCV，而是自己编译的呢？
 自己编译的OpenCV 应该也会有 pkg-config 的配置文件，可以使用pkg-config。如果实在没有， 也没问题。直接在Qt的工程文件里面添加OpenCV的头文件地址和 库文件地址就是了，Windows下的配置就是这么进行的 。
 
    win32{ 
        INCLUDEPATH += "C:/OpenCV/include"
        LIBS += -L"C:/OpenCV/lib" -lopencv_core -lopencv_imgproc -lopencv_highgui   -lopencv_ml -lopencv_video -lopencv_features2d -lopencv_calib3d -lopencv_objdetect -lopencv_contrib -lopencv_legacy -lopencv_flann
    } 
    
上面的C：/OpenCV/lib 等请改成你自己实际的OpenCV的安装地址。

具体的Qt的工程文件的语法，可以在Qt Creator的帮助文档里面找到，你搜 qmake project files 即可。


同样的，上面讲的是在Qt下写命令行的OpenCV程序，你也可以使用同样的方式在Qt的图形界面
程序中连接OpenCV的库并使用OpenCV的函数，这个我下次会讲一下。

## 参考文献

在学习OpenCV过程中有很多的文档可以学习，不过网上很多文档可能是针对比较老的版本，而且
使用的是C语言的接口，如果想了解比较新的版本和C++接口的 首选的是以下几本参考文献。

[OpenCV tutorials](http://docs.opencv.org/doc/tutorials/tutorials.html)

[master opencv with practical computer vision projects](https://www.google.com.hk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=4&cad=rja&ved=0CEkQFjAD&url=%68%74%74%70%3a%2f%2f%64%65%63%61%2e%63%75%63%2e%65%64%75%2e%63%6e%2f%43%6f%6d%6d%75%6e%69%74%79%2f%6d%65%64%69%61%2f%70%2f%33%32%38%36%33%2f%64%6f%77%6e%6c%6f%61%64%2e%61%73%70%78&ei=p9FwUrm7MIrHkQXb5IHYBA&usg=AFQjCNGa2iWtxy2AzkF-MXUArUOR9nzgmA&sig2=y3x9ByAjsZzUhI-YJcFTyQ)

[OpenCV_2_Computer_Vision_Application_Programming_Cookbook](https://www.google.com.hk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&ved=0CCoQFjAA&url=%68%74%74%70%3a%2f%2f%7a%65%6e%69%74%68%6c%69%62%2e%67%6f%6f%67%6c%65%63%6f%64%65%2e%63%6f%6d%2f%73%76%6e%2f%74%72%75%6e%6b%2f%62%6f%6f%6b%73%2f%4f%70%65%6e%43%56%5f%32%5f%43%6f%6d%70%75%74%65%72%5f%56%69%73%69%6f%6e%5f%41%70%70%6c%69%63%61%74%69%6f%6e%5f%50%72%6f%67%72%61%6d%6d%69%6e%67%5f%43%6f%6f%6b%62%6f%6f%6b%2e%70%64%66&ei=HNFwUtWlAoL5kgW-ooG4Dw&usg=AFQjCNGE_l6oDgAErQEsHH6QLTvBfW1ORw&sig2=f0qZ6uhAwmJY6xysf4GHFQ)

[OpenCV 在线文档](http://docs.opencv.org/)

在本文写作过程中，我也参考了以下一些文档

[理解pkg-config](http://www.chenjunlu.com/2011/03/understanding-pkg-config-tool/)

[qmake Project Files]() Qt帮助文档

[Qt官网](http://qt-project.org/)


