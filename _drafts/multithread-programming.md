---
layout: post
title: "C++多线程编程"
description: ""
category: 
tags: [multi-thread, programming]
---
{% include JB/setup %}

参考资料: [维基百科](https://en.wikipedia.org/wiki/Thread_(computing)#Multithreading)

## 历史



## 库

* POSIX Thread
* C++11 Thread
* Qt Thread

## 基本功能 API

* 创建线程

  pthread_create  std::thread  QThread()

* 阻塞线程

  pthread_join       thread::join  QThread::wait

* 分离线程

  pthread_detach   thread::detach  

## 同步机制

* mutex

* condition

  

## 使用多线程中遇到的问题

### 多线程与信号

 信号是类 Unix 系统中的一种进程间通信方法。众所周知，Unix 1969年诞生于贝尔实验室，而用户态的多线程直到 90年代才开始出现，所以，信号在设计之时是不可能知道怎么与多线程程序一同工作的。

在有些 linux 程序中，需要使用驱动程序向应用程序主动发送消息。有一种做法就是应用程序注册一个 SIGIO 的信号处理函数，然后驱动程序向应用程序发送 SIGIO 信号。

###  多线程与第三方库

 [Zint Barcode Generator](http://zint.github.io/) 是一个生成各种类型条码和二维码的开源库，它的代码并不是多线程安全的，在多线程环境下使用该库生成条码时，有可能会出现程序错误。

出现问题的代码就在 reedsol.c 这个文件中，它定义了三个全局变量，当多个线程同时访问 reedsol.c 中的函数时，会错误地改写这三个变量的值，当程序调用 rs_free() 函数时，则会重复释放已经被释放过的地址，导致程序出错。

[https://github.com/zint/zint/blob/3432bc9aff311f2aea40f0e9883abfe6564c080b/backend/reedsol.c](https://github.com/zint/zint/blob/3432bc9aff311f2aea40f0e9883abfe6564c080b/backend/reedsol.c)

```c
int *logt = NULL, *alog = NULL, *rspoly = NULL;
```

```c
void rs_init_gf(int poly)
{
	int m, b, p, v;

	// Find the top bit, and hence the symbol size
	for (b = 1, m = 0; b <= poly; b <<= 1)
		m++;
	b >>= 1;
	m--;
	gfpoly = poly;
	symsize = m;

	// Calculate the log/alog tables
	logmod = (1 << m) - 1;
	logt = (int *)malloc(sizeof(int) * (logmod + 1));
	alog = (int *)malloc(sizeof(int) * logmod);

	for (p = 1, v = 0; v < logmod; v++) {
		alog[v] = p;
		logt[p] = v;
		p <<= 1;
		if (p & b)
			p ^= poly;
	}
}
```

```c
void rs_init_code(int nsym, int index)
{
	int i, k;

	rspoly = (int *)malloc(sizeof(int) * (nsym + 1));

	rlen = nsym;

	rspoly[0] = 1;
	for (i = 1; i <= nsym; i++) {
		rspoly[i] = 1;
		for (k = i - 1; k > 0; k--) {
			if (rspoly[k])
				rspoly[k] = alog[(logt[rspoly[k]] + index) % logmod];
			rspoly[k] ^= rspoly[k - 1];
		}
		rspoly[0] = alog[(logt[rspoly[0]] + index) % logmod];
		index++;
	}
}
```

```c
void rs_free(void)
{ /* Free memory */
	free(logt);
	free(alog);
	free(rspoly);
	rspoly = NULL;
}
```

最简单的解决办法就是使用 mutex 将这三个变量保护起来，使多线程访问这几个变量串行化。

```c
#include <mutex>
static std::mutex d_mutex;

int *logt = NULL, *alog = NULL, *rspoly = NULL;
```

```c
void rs_init_gf(int poly)
{
	int m, b, p, v;
    
	// Find the top bit, and hence the symbol size
	for (b = 1, m = 0; b <= poly; b <<= 1)
		m++;
	b >>= 1;
	m--;
	gfpoly = poly;
	symsize = m;

	// Calculate the log/alog tables
	logmod = (1 << m) - 1;
    
    	d_mutex.lock();
    
	logt = (int *)malloc(sizeof(int) * (logmod + 1));
	alog = (int *)malloc(sizeof(int) * logmod);

	for (p = 1, v = 0; v < logmod; v++) {
		alog[v] = p;
		logt[p] = v;
		p <<= 1;
		if (p & b)
			p ^= poly;
	}
}
```

```c
void rs_free(void)
{ /* Free memory */
	free(logt);
	free(alog);
	free(rspoly);
	rspoly = NULL;
    
    	d_mutex.unlock();
}
```

