---
comments: true
date: 2011-12-01 21:08:00
layout: post
title: 写自己的头文件  ——C语言的多文件编译
categories:
- Linux
- ProgramminginC
tags:
- C
---

{% include JB/setup %}
每个学习C语言的人 肯定都知道程序的第一行要写上一个 

    #include <stdio.h> 

包含一个头文件,如果你有兴趣 可以去你的编译器的安装目录下找到这个文件 看看这个文件里有什么内容  
这里也有一个gcc 的stdio.h文件  

    http://115.com/file/bhy6k2yl#stdio.h?  
  
如果你自己写的程序很大 有很多的函数 ，那么你要怎么处理呢？ 全部写在一个main.c文件里面？ 肯定不是个好办法  
一般是写在不同的源文件里面，然后写个头文件 ，要使用源文件里的函数 只要在文件前面加个#include "your_head_file.h"就可以了。  
  
例如：  
test.c 的内容是  

    #include <stdio.h>  
    void fun1()  
    {  
    do_something();  
    }  

再写一个test.h 内容是  

    #ifndef _TEST_H_  
    #define _TEST_H_  
    void fun1();  
    #endif  
  
然后在要调用 fun1();函数的主文件前面加一个  

    #include"test.h"_  

就可以直接使用这个函数了。  
  
  
如果你是用GCC编译器在给你的程序编译，那么要用下面的方法编译你的程序  
  


假设我们有下面这样的一个程序,源代码如下:

    /* main.c */

    #include "mytool1.h"

    #include "mytool2.h"

    int main(int argc,char **argv)

    {

    mytool1_print("hello");

    mytool2_print("hello");

    }

    /* mytool1.h */

    #ifndef _MYTOOL_1_H_

    #define _MYTOOL_1_H_

    void mytool1_print(char *print_str);

    #endif

    /* mytool1.c */

    #include "mytool1.h"

    void mytool1_print(char *print_str)

    {

    printf("This is mytool1 print %sn",print_str);

    }

    /* mytool2.h */

    #ifndef _MYTOOL_2_H_

    #define _MYTOOL_2_H_

    void mytool2_print(char *print_str);

    #endif

    /* mytool2.c */

    #include "mytool2.h"

    void mytool2_print(char *print_str)

    {

    printf("This is mytool2 print %sn",print_str);

    }

当然由于这个程序是很短的我们可以这样来编译

    gcc -c main.c

    gcc -c mytool1.c

    gcc -c mytool2.c

    gcc -o main main.o mytool1.o mytool2.o

这样的话我们也可以产生main程序?  
  
如果你是用SDCC 给单片机之类的小程序编译   
则 上面的用法是  

    sdcc -c mytool1.c  
    sdcc -c mytool2.c  
    sdcc main.c mytool1.rel mytool2.rel -o main  
  



 
