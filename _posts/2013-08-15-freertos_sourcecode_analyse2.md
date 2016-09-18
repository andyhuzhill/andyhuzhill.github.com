---
layout: post
title: "FreeRTOS系列[三]源代码分析 list"
description: ""
category: FreeRTOS
tags: 
- FreeRTOS
---
{% include JB/setup %}
* TOC
{:toc}
<div style="border-bottom: 1px solid #ccc;line-height: 1.3em;"></div>

FreeRTOS里面实现了一个链表用来存储任务信息，这个链表也可以被用户程序使用。

##链表是什么？
链表是一种物理存储单元上非连续、非顺序的存储结构，数据元素的逻辑顺序是通过链表中的指针链接次序实现的。链表由一系列结点（链表中每一个元素称为结点）组成，结点可以在运行时动态生成。每个结点包括两个部分：一个是存储数据元素的数据域，另一个是存储下一个结点地址的指针域。 相比于线性表顺序结构，链表比较方便插入和删除操作。

学习数据结构的时候，第一个可能就是学习的链表，学习C语言的指针部分的时候，也可能会提到链表。

## FreeRTOS里面链表

先看看头文件里面链表结构体的定义吧。

{%   highlight c %}
/*
 * Definition of the only type of object that a list can contain.
 */
struct xLIST_ITEM
{
	configLIST_VOLATILE portTickType xItemValue;	/*< The value being listed.  In most cases this is used to sort the list in descending order. */
	struct xLIST_ITEM * configLIST_VOLATILE pxNext;	/*< 指向链表中下一个元素的指针. */
	struct xLIST_ITEM * configLIST_VOLATILE pxPrevious;/*< 指向链表中上一个元素的指针. */
	void * pvOwner;									/*< 指向链表元素(一般是任务控制块(TCB))的指针.  There is therefore a two way link between the object containing the list item and the list item itself. */
	void * configLIST_VOLATILE pvContainer;			/*< 指向存储元素的链表的指针. */
};
typedef struct xLIST_ITEM xListItem;				/* For some reason lint wants this as two separate definitions. */

struct xMINI_LIST_ITEM
{
	configLIST_VOLATILE portTickType xItemValue;
	struct xLIST_ITEM * configLIST_VOLATILE pxNext;
	struct xLIST_ITEM * configLIST_VOLATILE pxPrevious;
};
typedef struct xMINI_LIST_ITEM xMiniListItem;

/*
 * Definition of the type of queue used by the scheduler.
 */
typedef struct xLIST
{
	configLIST_VOLATILE unsigned portBASE_TYPE uxNumberOfItems;
	xListItem * configLIST_VOLATILE pxIndex;		/*< Used to walk through the list.  Points to the last item returned by a call to pvListGetOwnerOfNextEntry (). */
	xMiniListItem xListEnd;							/*< List item that contains the maximum possible item value meaning it is always at the end of the list and is therefore used as a marker. */
} xList;

{%   endhighlight %}

下面有许多的宏定义，其中一个引起我特别注意的是下面这个宏

{%   highlight  c %}

/*
 * Access function to obtain the owner of the next entry in a list.
 *
 * The list member pxIndex is used to walk through a list.  Calling
 * listGET_OWNER_OF_NEXT_ENTRY increments pxIndex to the next item in the list
 * and returns that entries pxOwner parameter.  Using multiple calls to this
 * function it is therefore possible to move through every item contained in
 * a list.
 *
 * The pxOwner parameter of a list item is a pointer to the object that owns
 * the list item.  In the scheduler this is normally a task control block.
 * The pxOwner parameter effectively creates a two way link between the list
 * item and its owner.
 *
 * @param pxList The list from which the next item owner is to be returned.
 *
 * \page listGET_OWNER_OF_NEXT_ENTRY listGET_OWNER_OF_NEXT_ENTRY
 * \ingroup LinkedList
 */
#define listGET_OWNER_OF_NEXT_ENTRY( pxTCB, pxList ) \
{	\
xList * const pxConstList = ( pxList );			\
	/* Increment the index to the next item and return the item, ensuring */\
	/* we don't return the marker used at the end of the list.  */	\
	( pxConstList )->pxIndex = ( pxConstList )->pxIndex->pxNext;	\
	if( ( void * ) ( pxConstList )->pxIndex == ( void * ) &( ( pxConstList )->xListEnd ) )	\
	{	    \
		( pxConstList )->pxIndex = ( pxConstList )->pxIndex->pxNext;	\
	}		\
	( pxTCB ) = ( pxConstList )->pxIndex->pvOwner;		\
}

{%  endhighlight %}

是不是觉得这个宏定义看起来像一个函数呢？ 却是用宏定义的方式写的，为什么要这么做呢？
这个应该是为了效率的因素。因为调用函数的时候，需要从目前的指令地址跳转到子函数的地址
执行，然后再跳回来，这个过程也是需要耗费指令时间的，如果直接将代码插入到需要执行的地
方，就不需要跳来跳去了。其实对于C++的使用者或者是C99的使用者，可以利用内联函数的方式
实现同样的功能，FreeRTOS之所以使用宏来实现估计是为了兼容老式的C语言编译器。

链表的主要操作有如下几个：

1.  初始化链表。
2.  插入链表元素
3.  删除链表元素

这几项操作都是用函数的方式定义的。具体的代码我就不列举了，FreeRTOS的源代码有很好的注释，而且链表的操作是每一个讲到数据结构的书都会写的。
