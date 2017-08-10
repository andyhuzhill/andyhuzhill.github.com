---
layout: post
title: "FreeRTOS介绍与移植"
description: ""
category: FreeRTOS
tags:
- FreeRTOS
- OS
---

* TOC
{:toc}
<hr/>

最近在看一个实时嵌入式操作系统————FreeRTOS, 为什么看它呢？首先它是开源的，其次它的内核最小只需要三个文件 `task.c、list.c、queue.c`，加起来5000多行代码还有很多注释在里面。他的优点网上也有很多介绍的，我也就不多说了，感兴趣的可以去百度google一下。

## 源代码目录结构
从[FreeRTOS](http://www.freertos.org/)的官方网站可以下载到FreeRTOS的全部源代码。下载下来的压缩包的目录结构类似如下：

       FreeRTOSV7.4.0/
                     |--FreeRTOS/           //这里是FreeRTOS的源代码和示例工程
                     |          |--Demo     //示例工程
                     |          |--License  //许可证文件
                     |          |--Source/   //内核源代码
                     |          |        |--include //头文件
                     |          |        |--portable  //芯片相关的代码
                     |          |        \--readme.txt
                     |          \--readme.txt
                     |--FreeRTOS-Plus     // 这里是FreeRTOS的一些增强程序 比如文件系统、 网络层等等
                     \--readme.txt
                 
                         
                         
我们主要还是看FreeRTOS目录下的东西，移植FreeRTOS到一个平台上主要要改以下几个文件：
  
    FreeRTOSConfig.h    //主要的配置文件，可以用来裁剪部分不需要的功能
    portable/<Compiler>/<Platform>/port.c
    portable/<Compiler>/<Platform>/portmacro.h
    portable/<Compiler>/<Platform>/portasm.s
    
上面的Compiler是你使用的编译器名称，Platform是你使用的平台名称，那个汇编文件是可选的，因为有些编译器可以在C语言中嵌入汇编。

## 开始移植
  
开始移植之前，你可以先去Demo目录下看看，有没有与你的板子相符合的工程，可以直接拿过来使用，或者是找一个相近的，修改一下就可以使用了。

###  portmacro.h

先看看portmacro.h文件

{% highlight c %}    
/* Type definitions. */
#define portCHAR        char
#define portFLOAT       float
#define portDOUBLE      double
#define portLONG        long
#define portSHORT       short
#define ortSTACK_TYPE  unsigned portLONG
#define portBASE_TYPE   long

#if( configUSE_16_BIT_TICKS == 1 )
   typedef unsigned portSHORT portTickType;
   #define portMAX_DELAY ( portTickType ) 0xffff
#else
   typedef unsigned portLONG portTickType;
   #define portMAX_DELAY ( portTickType ) 0xffffffff
#endif
/*-----------------------------------------------------------*/ 

/* Architecture specifics. */
#define portSTACK_GROWTH            ( -1 )
#define portTICK_RATE_MS            ( ( portTickType ) 1000 / configTICK_RATE_HZ )      
#define portBYTE_ALIGNMENT          8
/*-----------------------------------------------------------*/ 

/* Scheduler utilities. */
extern void vPortYieldFromISR( void );
#define portYIELD()                 vPortYieldFromISR()
#define portEND_SWITCHING_ISR( xSwitchRequired ) if( xSwitchRequired ) vPortYieldFromISR()
/*-----------------------------------------------------------*/

/* Critical section management. */
      extern void vPortEnterCritical( void );
      extern void vPortExitCritical( void );
      extern unsigned long ulPortSetInterruptMask( void );
      extern void vPortClearInterruptMask( unsigned long ulNewMaskValue );
#define portSET_INTERRUPT_MASK_FROM_ISR()       ulPortSetInterruptMask()
#define portCLEAR_INTERRUPT_MASK_FROM_ISR(x)    vPortClearInterruptMask(x)
#define portDISABLE_INTERRUPTS()                ulPortSetInterruptMask()
#define portENABLE_INTERRUPTS()                 vPortClearInterruptMask(0)
#define portENTER_CRITICAL()                    vPortEnterCritical()
#define portEXIT_CRITICAL()                     vPortExitCritical()

{% endhighlight %}

首先是将一些数据类型定义为port开头的类型，而不是使用C语言自带的类型名。然后是定义一些硬件相关的宏：

    portBYTE_ALIGNMENT //貌似是分配任务堆栈空间用的宏定义
    portSTACK_GROWTH   //定义堆栈生长的方向，一般是向下生长的 定义为-1
    portTICK_RATE_MS   //这个是在用户程序中能用到的 表示Tick间隔多少ms
    portYIELD()        //实现任务切换
    portNOP()           //空操作
    portENTER_CRITICAL()  //进入临界区
    portEXIT_CRITICAL()  //退出临界区
    portENABLE_INTERRUPTS() //开中断
    portDISABLE_INTERRUPTS() //关中断

### port.c

port.c文件里面就是实现了上面的头文件中用的的几个函数：

    pxPortInitialiseStack()
    xPortStartScheduler()
    vPortEndScheduler()
    vPortYield()
    vPortTickInterrupt()
    
定义了几个全局变量：

    /* The priority used by the kernel is assigned to a variable to make access
    from inline assembler easier. */
    const unsigned long ulKernelPriority = configKERNEL_INTERRUPT_PRIORITY;

    /* Each task maintains its own interrupt status in the critical nesting
    variable. */
    static unsigned portBASE_TYPE uxCriticalNesting = 0xaaaaaaaa;
    
### FreeRTOSConfig.h
  
  接下来就是FreeRTOS的全局配置文件了。先来看一个示例的FreeRTOSConfig.h:
 
{% highlight c %}
#ifndef FREERTOS_CONFIG_H
#define FREERTOS_CONFIG_H

/* 是否采用抢占式调度器 */
#define configUSE_PREEMPTION			1           
/* 是否使用空闲任务 */
#define configUSE_IDLE_HOOK				0   
/* 是否使用心跳钩子函数 */
#define configUSE_TICK_HOOK				0   
/* 定义MCU内核工作频率 */
#define configCPU_CLOCK_HZ				( 100000000UL )
/* 时钟Tick的频率 */
#define configTICK_RATE_HZ				( ( portTickType ) 1000 )
/* 程序中可以使用的最大优先级 */
#define configMAX_PRIORITIES			( ( unsigned portBASE_TYPE ) 5 )
/* 任务堆栈的最小大小 */
#define configMINIMAL_STACK_SIZE		( ( unsigned short ) 90 )
/* 设置堆空间的大小，只有当程序中采用FreeRTOS提供的内存分配算法才用到 */
#define configTOTAL_HEAP_SIZE			( ( size_t ) ( 8 * 1024 ) )
/* 任务名称最大长度，包括末尾的NULL结束字节 */
#define configMAX_TASK_NAME_LEN			( 10 )
/* 如果要使用TRACE功能 */
#define configUSE_TRACE_FACILITY		0
/* 设为1 portTickType将定义为无符号16整型，否则为无符号32位整型 */
#define configUSE_16_BIT_TICKS			0
/*这个参数控制那些优先级与idle 任务相同的任务的行为，并且只有当内核被配置为抢占式任务调度时才有实际作用。
 * 内核对具有同样优先级的任务会采用时间片轮转调度算法。当任务的优先级高于idle任务时，各个任务分到的时间片是同样大小的。
 * 但当任务的优先级与idle任务相同时情况就有些不同了。当configIDLE_SHOULD_YIELD 被配置为1时，当任何优先级与idle 任务相同的任务处于就绪态时，idle任务会立刻要求调度器进行任务切换。这会使idle任务占用最少的CPU时间，但同时会使得优先级与idle 任务相同的任务获得的时间片不是同样大小的。因为idle任务会占用某个任务的部分时间片  */
#define configIDLE_SHOULD_YIELD			0
/* 程序中是否包含mutex相关代码 */
#define configUSE_MUTEXES				1
/* 队列注册表有两个作用，但是这两个作用都依赖于调试器的支持：
 * 1.        给队列一个名字，方便调试时辨认是哪个队列。
 * 2.        包含调试器需要的特定信息用来定位队列和信号量。
 * 如果你的调试器没有上述功能，哪个这个注册表就毫无用处，还占用的宝贵的RAM空间 */
#define configQUEUE_REGISTRY_SIZE		8
/* 是否检测堆栈溢出 */
#define configCHECK_FOR_STACK_OVERFLOW	2
/* 是否包含recursive mutex代码 */
#define configUSE_RECURSIVE_MUTEXES		1
#define configUSE_MALLOC_FAILED_HOOK	0
#define configUSE_APPLICATION_TASK_TAG	0
#define configUSE_COUNTING_SEMAPHORES	1

/* Co-routine definitions. */
/* 是否使用协程 */
#define configUSE_CO_ROUTINES 		0
/* 协程可以使用的优先级数量 */
#define configMAX_CO_ROUTINE_PRIORITIES ( 2 )

/* Software timer definitions. */
/* 是否包含软件定时器 */
#define configUSE_TIMERS				1
/* 软件定时器任务优先级 */
#define configTIMER_TASK_PRIORITY		( 2 )
/* 设置软件定时器任务中用到的命令队列长度 */
#define configTIMER_QUEUE_LENGTH		10
/* 设置软件定时器任务所需任务堆栈大小 */
#define configTIMER_TASK_STACK_DEPTH	( configMINIMAL_STACK_SIZE * 2 )

/* Set the following definitions to 1 to include the API function, or zero
to exclude the API function. */
#define INCLUDE_vTaskPrioritySet		1
#define INCLUDE_uxTaskPriorityGet		1
#define INCLUDE_vTaskDelete				1
#define INCLUDE_vTaskCleanUpResources	1
#define INCLUDE_vTaskSuspend			1
#define INCLUDE_vTaskDelayUntil			1
#define INCLUDE_vTaskDelay				1

/* Run time stats gathering definitions. */
#ifdef __ICCARM__
	/* The #ifdef just prevents this C specific syntax from being included in
	assembly files. */
	void vMainConfigureTimerForRunTimeStats( void );
	unsigned long ulMainGetRunTimeCounterValue( void );
#endif
#define configGENERATE_RUN_TIME_STATS	1
#define portCONFIGURE_TIMER_FOR_RUN_TIME_STATS() vMainConfigureTimerForRunTimeStats()
#define portGET_RUN_TIME_COUNTER_VALUE() ulMainGetRunTimeCounterValue()

/* Cortex-M specific definitions. */
#ifdef __NVIC_PRIO_BITS
	/* __BVIC_PRIO_BITS will be specified when CMSIS is being used. */
	#define configPRIO_BITS       		__NVIC_PRIO_BITS
#else
	#define configPRIO_BITS       		4        /* 15 priority levels */
#endif

/* The lowest interrupt priority that can be used in a call to a "set priority"
function. */
#define configLIBRARY_LOWEST_INTERRUPT_PRIORITY			0xf

/* The highest interrupt priority that can be used by any interrupt service
routine that makes calls to interrupt safe FreeRTOS API functions.  DO NOT CALL
INTERRUPT SAFE FREERTOS API FUNCTIONS FROM ANY INTERRUPT THAT HAS A HIGHER
PRIORITY THAN THIS! (higher priorities are lower numeric values. */
#define configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY	5

/* Interrupt priorities used by the kernel port layer itself.  These are generic
to all Cortex-M ports, and do not rely on any particular library functions. */
/* 决定内核使用的优先级 */
#define configKERNEL_INTERRUPT_PRIORITY 		( configLIBRARY_LOWEST_INTERRUPT_PRIORITY << (8 - configPRIO_BITS) )
/* !!!! configMAX_SYSCALL_INTERRUPT_PRIORITY must not be set to zero !!!!
See http://www.FreeRTOS.org/RTOS-Cortex-M3-M4.html. */
/*决定了可以调用API函数的中断的最高优先级。高于这个值的中断处理函数不能调用任何API 函数。 */
#define configMAX_SYSCALL_INTERRUPT_PRIORITY 	( configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY << (8 - configPRIO_BITS) )
	
/* Normal assert() semantics without relying on the provision of an assert.h
header file. */
#define configASSERT( x ) if( ( x ) == 0 ) { taskDISABLE_INTERRUPTS(); for( ;; ); }	
	
/* Definitions that map the FreeRTOS port interrupt handlers to their CMSIS
standard names. */
#define vPortSVCHandler         SVC_Handler
#define xPortPendSVHandler      PendSV_Handler
#define xPortSysTickHandler     SysTick_Handler

#endif /* FREERTOS_CONFIG_H */

{% endhighlight %}

上面的文件，我已经做了部分注释了。 一般改好这三个文件，就算成功的将FreeRTOS移植到了你的芯片平台上了，至于怎么使用FreeRTOS， 你可以去官网查看帮助文档，官方有写一个使用手册《Using_the_FreeRTOS_Real_Time_Kernel-A_Practical_Guide_opened》,这本手册本来是要收费的，不过网上也是可以找的到下载的，网上也有一个网友翻译的中文版，你也可以去网上找来，看一看。
