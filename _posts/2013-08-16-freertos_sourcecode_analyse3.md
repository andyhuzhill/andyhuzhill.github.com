---
layout: post
title: "FreeRTOS系列[四]源代码分析 task"
description: ""
category: FreeRTOS
tags: 
- FreeRTOS
---
{% include JB/setup %}

现在来看FreeRTOS中最重要的一个东西，那就是任务管理。这是使用FreeRTOS的基础，也是
理解基于FreeRTOS应用程序行为方式的基础。

##  task.h

我们首先来看task.h这个头文件。

    69 #ifndef INC_FREERTOS_H
    70     #error "include FreeRTOS.h must appear in source files before include t     ask.h"
    71 #endif

这几句是一个预处理命令，当你没有包含FreeRTOS.h这个头文件而直接包含task.h头文件的时候，编译器在编译的时候会抛出一个错误。

接下来是包含了list.h，因为任务调度的时候会用到[上一篇](/freertos/2013/08/15/freertos_sourcecode_analyse2/)介绍的那个链表数据结构。

然后是定义了各种任务管理所需要的数据类型

{%   highlight  c %}

typedef void * xTaskHandle;        //任务句柄
typedef enum
{
    eRunning = 0,
    eReady,
    eBlocked,
    eSuspended,
    eDeleted
} eTaskState;         //这里定义了任务可能处于的五种状态

{%   endhighlight %}

这里，我要讲一下各个任务状态的意义。
   
eRunning 顾名思义就是运行态，处于这个状态的任务正在运行，并占用全部的CPU资源。

eReady  就绪态，这种状态的任务并没有被堵塞（Blocked）也没有被挂起（Suspennd），它已经准备好了要运行，但是又不是运行状态。

eBlocked 堵塞态，一个任务正在等待一个事件（Event）发生，就处于堵塞态。通常是等待以下两种事件。
    
   1. 时间相关的事件。比如要延时多少多少秒。
   2. 同步事件。比如需要等待另一个任务完成什么之后再执行本任务等等。
 
[[eSuspended]] 挂起态，处于这个状态的任务不会被任务调度器调度，进入这个状态只有通过vTaskSuspend()这个API，而退出这个状态也只能使用vTaskResume()或vTaskResumeFromISR()。大部分的程序并不需要挂起态。

eDeleted  删除，就是这个任务已经从任务调度器的任务链表中删除了。

下面是一个状态转换图：

![](/images/freertos/taskstates.png)

## 任务控制块

任务控制块是FreeRTOS用来保存任务状态的一种数据结构。FreeRTOS每创建一个任务，就会分配一个任务控制块来描述该任务。

任务控制块的申明在task.c文件中， 如下

{% highlight c %}
typedef struct tskTaskControlBlock
{
	volatile portSTACK_TYPE	*pxTopOfStack;		/*< Points to the location of the last item placed on the tasks stack.  THIS MUST BE THE FIRST MEMBER OF THE TCB STRUCT. */

	#if ( portUSING_MPU_WRAPPERS == 1 )
		xMPU_SETTINGS xMPUSettings;				/*< The MPU settings are defined as part of the port layer.  THIS MUST BE THE SECOND MEMBER OF THE TCB STRUCT. */
	#endif

	xListItem				xGenericListItem;	/*< The list that the state list item of a task is reference from denotes the state of that task (Ready, Blocked, Suspended ). */
	xListItem				xEventListItem;		/*< Used to reference a task from an event list. */
	unsigned portBASE_TYPE	uxPriority;			/*< The priority of the task.  0 is the lowest priority. */
	portSTACK_TYPE			*pxStack;			/*< Points to the start of the stack. */
	signed char				pcTaskName[ configMAX_TASK_NAME_LEN ];/*< Descriptive name given to the task when created.  Facilitates debugging only. */

	#if ( portSTACK_GROWTH > 0 )
		portSTACK_TYPE *pxEndOfStack;			/*< Points to the end of the stack on architectures where the stack grows up from low memory. */
	#endif

	#if ( portCRITICAL_NESTING_IN_TCB == 1 )
		unsigned portBASE_TYPE uxCriticalNesting; /*< Holds the critical section nesting depth for ports that do not maintain their own count in the port layer. */
	#endif

	#if ( configUSE_TRACE_FACILITY == 1 )
		unsigned portBASE_TYPE	uxTCBNumber;	/*< Stores a number that increments each time a TCB is created.  It allows debuggers to determine when a task has been deleted and then recreated. */
		unsigned portBASE_TYPE  uxTaskNumber;	/*< Stores a number specifically for use by third party trace code. */
	#endif

	#if ( configUSE_MUTEXES == 1 )
		unsigned portBASE_TYPE uxBasePriority;	/*< The priority last assigned to the task - used by the priority inheritance mechanism. */
	#endif

	#if ( configUSE_APPLICATION_TASK_TAG == 1 )
		pdTASK_HOOK_CODE pxTaskTag;
	#endif

	#if ( configGENERATE_RUN_TIME_STATS == 1 )
		unsigned long ulRunTimeCounter;			/*< Stores the amount of time the task has spent in the Running state. */
	#endif

	#if ( configUSE_NEWLIB_REENTRANT == 1 )
		/* Allocate a Newlib reent structure that is specific to this task.
		Note Newlib support has been included by popular demand, but is not
		used by the FreeRTOS maintainers themselves.  FreeRTOS is not
		responsible for resulting newlib operation.  User must be familiar with
		newlib and must provide system-wide implementations of the necessary
		stubs. Be warned that (at the time of writing) the current newlib design
		implements a system-wide malloc() that must be provided with locks. */
		struct _reent xNewLib_reent;
	#endif

} tskTCB;
{% endhighlight %}


## 创建任务

要使用任务，第一步就是创建任务函数。任务函数要怎么写呢？ 下面是一个示例：

{% highlight c %}
void vTaskCode( void * pvParameters)
{
    for(;;)
    {
        // Task code goes here.
    }
}
{% endhighlight %}

任务函数的内容就是一个无限循环，有一个预定的参数pvParameters,而且没有返回值。无限
循环里面执行你的功能代码。

任务函数写好之后就要将它添加到任务调度器里面，这样才能真正运行。这时要调用FreeRTOS
的一个API——xTaskCreate()， 函数原型如下：
{% highlight c %}
portBASE_TYPE  xTaskCreate(
                    pdTASK_CODE pvTaskCode,
                    const char* const pcName,
                    unsigned short usStackDepth,
                    void *pvParameters,
                    unsigned portBASE_TYPE uxPriority,
                    xTaskHandle *pvCreatedTask
                    );
{% endhighlight %}

`pvTaskCode` 指向任务入口函数，就是任务函数名。

`pcName`   一个任务描述字符串，这个只对调试有用。

`usStackDepth`
告诉内核给任务分配多大的栈空间，他的大小不是字节，而是MCU的字
，比如在32位机器上，他分配的栈空间就是usStackDepth*4 字节。

`pvParameters` 这个就是传递给任务函数的参数。

`uxPriority` 任务优先级。

`pvCreatedTask` 任务函数的操作句柄。

实际上xTaskCreate函数是一个宏定义，最后实现功能的是xTaskGenericCreate()函数。

下面是一个典型的xTaskCreate的应用示例：
{% highlight c %}
int
main(void)
{
    xTaskHandle xHandle;     //声明一个任务句柄

    xTaskCreate( vTaskCode, "TaskName", STACK_SIZE, NULL, tskIDLE_PRIORITY,
    &xHandle);              //创建任务

    vTaskStartScheduler();   //启动任务调度器
    
    for(;;);

    return 0;
}

{%  endhighlight %}


## 

关于FreeRTOS的任务管理，还有很多东西，不可能在一篇blog里面讲完，就留待下次吧。

## 参考文献

Using the FreeRTOS Real Time Kernel
