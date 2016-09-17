---
layout: post
title: "FreeRTOS系列[二]源代码分析 FreeRTOS.h"
description: ""
category: FreeRTOS
tags: 
- FreeRTOS
---
{% include JB/setup %}

从这一篇开始，我就一点一点的把我在读FreeRTOS源代码中遇到的东西记录下来。


首先，我先说明我使用的是哪个版本，到目前为止（2013-08-15)最新版本的FreeRTOS是V7.5.2。

## 第一个头文件 FreeRTOS.h

第一个文件就是每一个使用了FreeRTOS的程序都需要包含的一个头文件————FreeRTOS.h。

FreeRTOS的源代码，每一个文件的文件头都有一大段的版权声明和帮助文档的链接。虽然很罗嗦，但是这还是一个比较好的编程习惯，一般的程序编辑器都是可以自动插入预先设定好的一段版权说明的。对于我们自己写的程序，不需要写这么长的说明，只要有如下几个要点就可以了。

1.  工程名和文件名
2.  文件的功能简要说明
3.  作者名
4.  版权协议声明
5.  程序的修改记录


首先看到的是第72行（我阅读的是官方下载的V7.5.2版本，如果你看到的行数和我的不同应该是因为版本不同）
这里使用尖括号"<>"包含了一个头文件

    #include <stddef.h>

尖括号说明这个文件是要在编译器的默认路径下搜索，可以看到stddef.h文件并不是FreeRTOS源代码里面提供的文件。不同的编译器stddef.h里面的内容也不同，不过一般都是和标准C的一些规定有关的设置。

接下来第75行是

    #include "projdefs.h"

这个头文件是FreeRTOS的一些基本设定，主要定义了如下一些宏定义

    pdTASK_CODE   //任务函数原型类型
    pdFALSE
    pdTRUE
    pdPASS
    pdFAIL
    errQUEUE_EMPTY
    errQUEUE_FULL
    errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY
    errNO_TASK_TO_RUN
    errQUEUE_BLOCKED
    errQUEUE_YIELD


第78行

    #include "FreeRTOSConfig.h"

之前我的[博文](/freertos/2013/07/30/freertostransplant)介绍了这个移植的时候要修改的FreeRTOS的全局配置文件。

第87行

    #include "portable.h"

这个头文件的前半部分其实是包含之前移植FreeRTOS的时候写的portmacro.h文件。portable.h文件的第351行包含了一个头文件 `"mpu_wrappers.h"`,这个头文件是干嘛的呢？从文件名看，MPU是ARM体系结构里面的一个东西全称是Memory Protection Unit,内存保护单元，wrapper是封装的意思，这个头文件将一些函数重定义为了使用MPU的函数，如果定义了使用MPU，还会定义了两个宏

{% highlight c %}
/* Ensure API functions go in the privileged execution section. */
#define PRIVILEGED_FUNCTION __attribute__((section("privileged_functions")))
#define PRIVILEGED_DATA __attribute__((section("privileged_data")))
{% endhighlight %}
    
这两个宏的目的是将API函数定义为链接到privileged execution section。如果没有定义使用MPU，则这两个宏都会定义为空的。

portable.h 文件还声明了几个与任务、内存管理相关的函数原型

{%  highlight c %}

#if( portUSING_MPU_WRAPPERS == 1 )
    portSTACK_TYPE *pxPortInitialiseStack( portSTACK_TYPE *pxTopOfStack, pdTASK_CODE pxC    ode, void *pvParameters, portBASE_TYPE xRunPrivileged ) PRIVILEGED_FUNCTION;
#else
    portSTACK_TYPE *pxPortInitialiseStack( portSTACK_TYPE *pxTopOfStack, pdTASK_CODE pxC    ode, void *pvParameters );
#endif

/*
 * Map to the memory management routines required for the port.
 */
void *pvPortMalloc( size_t xSize ) PRIVILEGED_FUNCTION;
void vPortFree( void *pv ) PRIVILEGED_FUNCTION;
void vPortInitialiseBlocks( void ) PRIVILEGED_FUNCTION;
size_t xPortGetFreeHeapSize( void ) PRIVILEGED_FUNCTION;

/*
 * Setup the hardware ready for the scheduler to take control.  This generally
 * sets up a tick interrupt and sets timers for the correct tick frequency.
 */
 portBASE_TYPE xPortStartScheduler( void ) PRIVILEGED_FUNCTION;
 
/*
 * Undo any hardware/ISR setup that was performed by xPortStartScheduler() so
 * the hardware is left in its original condition after the scheduler stops
 * executing.
 */
void vPortEndScheduler( void ) PRIVILEGED_FUNCTION;

{%  endhighlight %}

再接下来回到FreeRTOS.h文件

    typedef portBASE_TYPE  (*pdTASK_HOCK_CODE)(void*);

又定义了一个任务钩子函数原型的类型。

再接下来的文件内容就是检查所有必须的宏定义是否已经在FreeRTOSConfig.h里面被定义了，还有就是给一些未定义的宏设置一个默认值。


以上就是FreeRTOS.h文件的全部内容。

