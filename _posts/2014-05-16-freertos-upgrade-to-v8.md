---
layout: post
title: "FreeRTOS 升级到了V8.0版本"
description: ""
category: FreeRTOS
tags: 
- FreeRTOS 
---
{% include JB/setup %}
* TOC
{:toc}
<hr/>


最近感叹开源软件的版本更新速度非常之快，Linux内核最新版已经到了3.14.4, Qt去年年底才发布了Qt5.2，现在5.3马上也要发布了。连微软的Windows也学习了加快版本更新，以后Windows系统也许不会有Server Pack了，而是采用Update的方式更新。


回到我今天要说的主角-FreeRTOS, 之前我写过几篇博客，当时的FreeRTOS最新版是V7.5.0, 现在它也已经迈入到了V8.0时代。下面我来说说V8.0有什么改变。

## 新的特性-Event Group

FreeRTOS V8包含了一个全新的特性-Event Group。一个新的源文件 event_groups.c 实现了该特性。当然这个特性也是可选的，只有当你的程序需要这个特性的时候才需要包含这个文件。。


## 集中的延迟性中断处理

理想情况下一个中断服务程序应该是尽量保持短小，但是有时候在中断服务程序中确实需要处理一大堆事情，或者是需要执行一个不确定的任务。以前的做法是使用一个信号量在中断中解除一个高优先级的应用人物的堵塞，将实际的处理工作延迟到应用任务中。FreeRTOS允许中断直接返回到未阻塞的任务来保证处理的实时性。

FreeRTOS V8 提供了一种集中化的延迟中断处理机制，允许RTOS守护任务（通常称为时钟守护任务）来实现延迟的处理，避免了用户自己去实现延迟中断机制的负担。

文档参见以下链接：

[xTimerPendFunctionCallFromISP() APIO文档](http://www.freertos.org/xTimerPendFunctionCallFromISR.html)


## 使用 stdint.h

除了字符指针，所有的标准C类型都使用stdint.h中的相对应的类型typedef替换了。例如 uint32_t 替换了 unsigned long, int16_t  替换了short等等。

## 字符指针

字符指针用来引用一个文本字符串，比如分配给任务的名字。

过去，而且出于一个好的软件工程的实践，FreeRTOS代码规范不允许未显式指明有符号或无符号的char类型。因此，当使用标准字符串处理函数时，字符指针引用字符串时会需要强制类型转换。 这个强制类型转换保证了编译器默认使用无符号char型或编译器默认使用有符号char型时都不会产生编译警告。

不过后来的MISRA标准放宽了该要求，未显式指明的char类型被允许使用当且仅当：

× char 是用来指向一个人类阅读的字符串。
× char 用来保存单个的ASCII字符。

## 追踪与可视化

Trace 功能已经被增强，并且包括了堆使用率和新的event groups特性。
  
注意： FreeRTOS V8.0.0 需要 FreeRTOS+Trace 2.6或以后的版本

## 新定义的FreeRTOS类型名

为了保持与新的FreeRTOS和软件组件的连续性，提供与stdint.h定义类型的一致性，一些核心代码的类型名称已经改变了，所有的类型名目前都已经以“_t"结尾。

注意：为了保持向下兼容性，代码中仍可以使用旧的名字仍可使用，但强烈不推荐使用旧名称。（参加 [configENABLE_BACKWARD_COMPATIBILTY](http://www.freertos.org/a00110.html#configENABLE_BACKWARD_COMPATIBILITY) 配置项目）

<table>
<tr>
    <td> 旧的名称 </td>
    <td> 新的名称 </td>
    <td> 描述     </td>
</tr>
<tr>
    <td> xTaskHandle  </td>
    <td> TaskHandle_t </td>
    <td> 用来引用任务 </td>
</tr>
<tr>
    <td> xQueueHandle  </td>
    <td> QueueHandle_t </td>
    <td> 用于引用队列  </td>
</tr>
<tr>
    <td> xSemaphoreHandle </td>
    <td> SemaphoreHandle_t </td>
    <td> 用于引用二值、计数、递归和互斥锁类型信号量     </td>
</tr>
<tr>
    <td> xTimerHandle </td>
    <td> TimerHandle_t </td>
    <td> 用于引用软件定时器     </td>
</tr>
<tr>
    <td> xCoRoutineHandle </td>
    <td> CoRoutineHandle_t </td>
    <td> 用于引用协程     </td>
</tr>
<tr>
    <td> portTickType </td>
    <td> TickType_t </td>
    <td> 用于保存心跳计数值     </td>
</tr>
<tr>
    <td> portBASE_TYPE </td>
    <td> BaseType_t </td>
    <td> 用于保存该架构最有效率的有符号类型     </td>
</tr>
<tr>
    <td> unsigned portBASE_TYPE </td>
    <td> UBaseType_t </td>
    <td> 用于保存该架构最有效率的无符号类型    </td>
</tr>
<tr>
    <td> xQueueSetHandle </td>
    <td> QueueSetHandle_t </td>
    <td> 用于引用 queue set     </td>
</tr>
<tr>
    <td> xQueueSetMemberHandle </td>
    <td> xQueueSetMemberHandle_t </td>
    <td> 用于引用一个 queue set成员.可以是队列或其他任何信号量类型 </td>
</tr>
<tr>
    <td> xMemoryRegion </td>
    <td> MemoryRegion_t </td>
    <td> 用于支持内存保护架构的移植    </td>
</tr>
<tr>
    <td> xTaskParameters </td>
    <td> TaskParameters_t </td>
    <td> 用于任务函数参数    </td>
</tr>
<tr>
    <td> xTaskStatusType </td>
    <td> TaskStatus_t </td>
    <td> 供uxTaskGetSytemState()函数使用     </td>
</tr>
<tr>
    <td> pdTASK_HOOK_CODE </td>
    <td> TaskHookFunction_t </td>
    <td> 供任务标签函数使用 例如vTaskSetApplicationTag()函数  </td>
</tr>
<tr>
    <td> pdTASK_CODE </td>
    <td> TaskFunction_t </td>
    <td> 供 xTaskCreate()函数使用     </td>
</tr>
<tr>
    <td> tmrTIMER_CALLBACK </td>
    <td> TimerCallbackFunction_t </td>
    <td> 供 xTimerCreate()函数使用     </td>
</tr>
<tr>
    <td> xTimeOutType </td>
    <td> TimeOut_t </td>
    <td> 仅供高级用户使用     </td>
</tr>
</table>



### 

以上内容翻译自 [FreeRTOS官方网站](http://www.freertos.org/upgrading-to-FreeRTOS-V8.html)
