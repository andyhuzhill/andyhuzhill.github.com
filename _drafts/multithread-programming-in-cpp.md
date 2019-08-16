---
layout: post
title: "在C++中使用多线程编程"
description: ""
category: 
tags: [multi-thread, programming, cpp]
---
{% include JB/setup %}

参考资料: [维基百科-多线程](https://en.wikipedia.org/wiki/Thread_(computing)#Multithreading)

参考资料: [C++ 多线程系统编程精要-陈硕](https://blog.csdn.net/solstice/article/details/6181488)

参考资料: [C++ Concurrency in Action, 2nd Edition](https://www.manning.com/books/c-plus-plus-concurrency-in-action-second-edition)

参考资料: [Thread Support in Qt](https://doc.qt.io/qt-5/threads.html)


## 概念
线程是操作系统任务调度的最小单位，进程是一个运行中的程序。

最早的操作系统并没有用户态的线程，一个程序想要实现并发操作，就得靠多进程或者是IO复用的事件驱动编程。

多进程之间的内存地址空间是相互隔离的，如果想要交换数据的话，就不能直接传递一个指针让另一个进程去读取，得通过各种各样的进程间通信机制进行。

多线程的程序则是在同一个进程里面运行的，共享地址空间。共享地址空间好处就是避免了进程间通信的开销，但是同时也带来了许许多多的问题。
 

## 库
本文主要讲解 Linux 平台下的多线程编程，主要使用以下三种库作为讲解的例子。

* POSIX Thread

  Linux 平台上事实上的多线程库标准。

* C++11 std::thread

  C++11 标准新添加的多线程标准库，在 Linux 平台使用时，实际底层是使用 pthread 实现的 std::thread，所以在链接时需要加上 -lpthread .

* Qt QThread

  跨平台的具有事件循环和信号与槽功能的多线程库。


其实想要使用多线程编程，并不需要掌握太多的东西，只需要掌握几个创建和使用线程的 API，然后再掌握几种同步线程的机制就够用了。

要用好多线程，关键是要熟悉多线程的执行机制和原理，了解各种可能发生的问题和解决方法。


## 基本功能 API

* 创建线程

POSIX Thread

```c
#include <pthread.h>
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                          void *(*start_routine) (void *), void *arg);
```
使用 [pthread_create](http://man.he.net/?topic=pthread_create&section=all) 创建一个线程，其中 start_routine 就是要在新线程中执行的函数，arg 是传递给这个函数的参数。详细的说明可以通过 [`man pthread_create`](http://man.he.net/?topic=pthread_create&section=all) 命令查看。

C++11 std::thread

```cpp
#include <thread>

thread() noexcept;

thread( thread&& other ) noexcept;

template< class Function, class... Args > 
explicit thread( Function&& f, Args&&... args );
```

C++11 新添加的多线程标准库，使用方法也很简单，直接将要在新线程中执行的函数作为参数传递给 thread 的构造函数即可。这个函数既可以是一个函数指针，也可以是一个重载了 operator() 的对象，还可以是 C++11 新加入的 lambda 函数。

Qt QThread

```cpp
#include <QThread>

QThread::QThread(QObject *parent = nullptr)
```
在 Qt 中创建线程有两种方式

一种是创建一个派生自 [QObject](https://doc.qt.io/qt-5/qobject.html) 的工作类，然后使用 [QObject::moveToThread()](https://doc.qt.io/qt-5/qobject.html#moveToThread) 将这个对象移动到工作线程中，然后连接信号与工作类中的槽函数。

```cpp
class Worker : public QObject
{
	Q_OBJECT

public slots:
	void doWork(const QString &parameter) {
		QString result;
		/* ... here is the expensive or blocking operation ... */
		emit resultReady(result);
	}

signals:
	void resultReady(const QString &result);
};

class Controller : public QObject
{
	Q_OBJECT
	QThread workerThread;
public:
	Controller() {
		Worker *worker = new Worker;
		worker->moveToThread(&workerThread);
		connect(&workerThread, &QThread::finished, worker, &QObject::deleteLater);
		connect(this, &Controller::operate, worker, &Worker::doWork);
		connect(worker, &Worker::resultReady, this, &Controller::handleResults);
		workerThread.start();
	}
	~Controller() {
		workerThread.quit();
		workerThread.wait();
	}
public slots:
	void handleResults(const QString &);
signals:
	void operate(const QString &);
};
```

另一种是直接创建派生自 [QThread](https://doc.qt.io/qt-5/qthread.html) 的类，并重写 [QThread::run()](https://doc.qt.io/qt-5/qthread.html#run) 函数。
```cpp
class WorkerThread : public QThread
{
	Q_OBJECT
	void run() override {
		QString result;
		/* ... here is the expensive or blocking operation ... */
		emit resultReady(result);
	}
signals:
	void resultReady(const QString &s);
};

void MyObject::startWorkInAThread()
{
	WorkerThread *workerThread = new WorkerThread(this);
	connect(workerThread, &WorkerThread::resultReady, this, &MyObject::handleResults);
	connect(workerThread, &WorkerThread::finished, workerThread, &QObject::deleteLater);
	workerThread->start();
}
```

这两种方式的区别就是：

1.第一种方式会在工作线程中执行一个 [QEventLoop](https://doc.qt.io/qt-5/qeventloop.html), 第二种除非调用 [QThread::exec()](https://doc.qt.io/qt-5/qthread.html#exec) 是不会调用 [QEventLoop](https://doc.qt.io/qt-5/qeventloop.html) 的。

2.第一种方式，工作类的槽函数是在新建的线程中运行，而第二种方式只有 [QThread::run()](https://doc.qt.io/qt-5/qthread.html#run) 函数中的代码是在新建的线程中运行，它的槽函数还是在旧的线程中运行。


* 等待线程结束
	
使用上面的API创建线程之后，程序就会自动提交给线程库去调度执行了，但是我们是不知道程序具体是什么时候开始执行，也不知道线程什么时候结束的。这时我们就可以使用下面的函数来等待线程执行结束。

POSIX Thread
   
```c
#include <pthread.h>

int pthread_join(pthread_t thread, void **retval);
```
C++11 std::thread

[std::thread::join](https://en.cppreference.com/w/cpp/thread/thread/join)

```cpp
void join();
```

Qt QThread

[QThread::wait](https://doc.qt.io/qt-5/qthread.html#wait)

```cpp
bool QThread::wait(unsigned long time = ULONG_MAX);
```
   QThread 还可以通过 [QThread::finished()](https://doc.qt.io/qt-5/qthread.html#finished) 信号知道线程执行结束。

* 分离线程

  如果我们不需要等待线程结束，启动一个线程之后，就让它在后台一直运行，那么我们可以使用 detach 函数。

POSIX Thread

```c
#include <pthread.h>

int pthread_detach(pthread_t thread);
```

C++11 std::thread

[std::thread::detach](https://en.cppreference.com/w/cpp/thread/thread/detach)

```cpp
#include <thread>
void detach();
```


## 线程同步机制

上面介绍了如何创建线程，以及如何等待线程结束或者是让线程在后台运行。接下来，介绍一下线程同步的机制。我们先来看一个问题，如果没有线程同步会发生什么呢？

```cpp
#include <thread>
#include <iostream>
using namespace std;

int global = 0;

void func1()
{
    for ( int i = 0; i < 1000; ++i ) {
        global += 1;
    }
}

void func2()
{
    for ( int i = 0; i < 1000; ++i ) {
        global -= 1;
    }
}

int main(int argc, char *argv[])
{
    thread t1(func1);
    thread t2(func2);

    t2.join();
    t1.join();

    cout << " global = " << global << endl;
    return 0;
}
```

使用如下命令编译
```bash
g++ -std=c++11 -o test test.cpp -lpthread
```
执行多次程序，你会发现每次程序输出的结果都不太一样，并不如我们所预想的那样每次都输出0. 为什么呢，我们要知道，我们启动的两个线程是同时运行的，


* mutex

## 使用多线程中遇到的问题
 
 多线程编程中遇到的问题基本上可以分为以下几种类型:
 
 1. 死锁 (dead-lock)

    线程A要依次获取 mutex1 和 mutex2 去执行下一步操作，而线程B需要依次获取 mutex2 和 mutex1 去执行下一步操作。线程A获取了 mutex1， 线程B获取了 mutex2, 双方都要去获取对方已经 lock 的 mutex时，就会发生死锁。

	还有一种情况就是在同一个线程中，重复去 lock 已经 lock 的 mutex，这时也会发生死锁。（这里的mutex都是非递归mutex)

	要避免死锁的话，就是获取多个锁的时候，总是使用同一个顺序去访问，就能避免了。

	C++11 提供了一个 [std::lock](https://en.cppreference.com/w/cpp/thread/lock) 函数，用于同时获取多个锁时，避免死锁，你可以点击 [std::lock](https://en.cppreference.com/w/cpp/thread/lock) 链接去了解具体的使用方法。


 2. 竞态条件 (race-condition)


下面介绍两个在多线程程序中遇到的问题和解决方法。

### 多线程与信号

信号是类 Unix 系统中的一种进程间通信方法。众所周知，Unix 1969年诞生于贝尔实验室，而用户态的多线程直到 90年代才开始出现，所以，信号在设计之时是不可能知道怎么与多线程程序一同工作的，所以在多线程程序中使用信号，需要特别小心，否则就会出现意想不到的错误。

在有些 Linux 程序中，需要使用驱动程序向应用程序主动发送消息。有一种做法就是应用程序注册一个 SIGIO 的信号处理函数，然后驱动程序向应用程序发送 SIGIO 信号，信号处理函数接收到信号后就会被执行，这种处理方式完全是异步的，我们不知道什么时候信号处理函数会被调用。在信号处理函数中有许多限制，有些系统调用是无法使用的，例如 read, recv 因为它们会被异步的信号中断。应用程序对于 SIGIO 信号的默认操作是终止进程。
 
详细的 Linux 系统上关于信号机制的说明，可以通过命令 [`man 7 signal`](http://man.he.net/man7/signal) 查看。

那么如何在多线程程序中正确的处理信号呢？

基本方法就是将异步的信号调用转换成同步调用。首先，在主线程中屏蔽 SIGIO 信号，然后在需要处理 SIGIO 信号的线程中使用 sigwait 函数阻塞地等待信号的发生。代码如下所示：

主线程屏蔽信号

```cpp
#include <signal.h>

int main(int argc, char *argv[]) 
{
    sigset_t bset;
    
    sigemptyset(&bset);
    sigaddset(&bset, SIGIO); // 设置要屏蔽的信号
    
    if (pthread_sigmask(SIG_BLOCK, &bset, nullptr) != 0) {
        perror("set sig mask error!");
    }

    // 省略以下代码
}
```

工作线程中等待信号发生

```cpp
#include <signal.h>

void run() // 工作线程
{
    sigset_t bset;
    sigemptyset(&bset);
    sigaddset(&bset, SIGIO); // 注册等待的信号

    for (;;) {
        int sig = 0;
        int rc = sigwait(&bset, &sig); // 等待信号发生

        if (!rc) { // 有信号发生
            // 省略工作代码
        }
    }
}
```

###  多线程与第三方库

 [Zint Barcode Generator](http://zint.github.io/) 是一个生成各种类型条码和二维码的开源库，它的代码并不是多线程安全的，在多线程环境下使用该库生成条码时，有可能会出现程序错误。

出现问题的代码就在 [reedsol.c](https://github.com/zint/zint/blob/3432bc9aff311f2aea40f0e9883abfe6564c080b/backend/reedsol.c) 这个文件中，它定义了三个全局变量，当多个线程同时访问 [reedsol.c](https://github.com/zint/zint/blob/3432bc9aff311f2aea40f0e9883abfe6564c080b/backend/reedsol.c) 中的函数时，会错误地改写这三个变量的值，当程序调用 [rs_free()](https://github.com/zint/zint/blob/3432bc9aff311f2aea40f0e9883abfe6564c080b/backend/reedsol.c#L154-L160) 函数时，则会重复释放已经被释放过的地址，导致程序出错。

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

```diff
+ #include <mutex>
+ static std::mutex d_mutex;

int *logt = NULL, *alog = NULL, *rspoly = NULL;
```

```diff
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
    
+	d_mutex.lock();
    
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

```diff
void rs_free(void)
{ 
    /* Free memory */
	free(logt);
	free(alog);
	free(rspoly);
	rspoly = NULL;
    
+	d_mutex.unlock();
}
```

## 帮助多线程编程的工具

参考资料 [C/C++ 线程安全分析](https://static.googleusercontent.com/media/research.google.com/zh-CN//pubs/archive/42958.pdf)

上面的参考资料介绍了一种“线程安全注解”的工具，给多线程代码加上注解，可以让编译器在忘记加锁的代码处给出警告。
