---
layout: post
title: "篮球比赛计时记分器"
date: 2011-12-11
categories:
- 80c51
- C51
tags:
- 80c51
- 小作品
---
{% include JB/setup %}
* TOC
{:toc}
<div style="border-bottom: 1px solid #ccc;line-height: 1.3em;"></div>

这里，我把之前参加学校比赛的一个小作品放在网上，虽然有很多错误，不过，希望还是能对一些人有帮助吧.

原理图如下：
![原理图](/images/jsq.jpg)

源程序如下：

{% highlight c %}
/* * =====================================================================
* *     Filename:  main.h
*    Description:  declaration for jsq
*        Version:  1.0
*       Created:  27.11.2011 11:51:54
*       Revision:  none
*       Compiler:  sdcc 3.0.0
*         Author:  YOUR NAME (), *        Company: * *
 ===================================================================== */
#ifndef _MAIN_H_
#define _MAIN_H_
#define uchar unsigned char
#define uint unsigned int
#define FEN 12
//定义蜂鸣器
__sbit  __at (0x96) fmq //sbit fmq=P1^6;
uchar  sz[] = {0xc0, 0xf9, 0xa4, 0xb0, 0x99, 0x92, 0x82, 0xf8, 0x80, 0x90};
//共阳极数字编码(0x0~0x9)
uchar szy[] = {0x3f, 0x06, 0x5b, 0x4f, 0x66, 0x6d, 0x7d, 0x07, 0x7f, 0x6f};
//共阴极数字段码//
uchar sw[] = {0xfe, 0xfd, 0xfb, 0xf7, 0xef, 0xdf, 0xbf, 0x7f};
uchar sw[] = {0x25, 0x26, 0x20, 0x2c, 0x34, 0x24, 0x64, 0xa4};
//位码(第0位至第7位)
//定义按键
__sbit __at (0xB0)  K1 ; //A队加一分 * sbit K1=P3^0;
__sbit __at (0xB1)  K2 ; //A队减一分 sbit K2=P3^1;
__sbit __at (0xB4)  K3 ; //B队加一分 sbit K3=P3^4;
__sbit __at (0xB5)  K4 ; //B队减一分 sbit K4=P3^5;
__sbit __at (0xB2)  K5 ; //24s重置   sbit K5=P3^2;
__sbit __at (0xB3)  K6 ; //比赛暂停  sbit K6=P3^3;
//定义计时器引脚
__sbit __at (0x93)  dscs ; //sbit dscs=P1^0;
__sbit __at (0x94)  dsas ;//sbit dsas=P1^1;
__sbit __at (0x95)  dsrw ;//sbit dsrw=P1^2;
__sbit __at (0x96)  dsds ;//sbit dsds=P1^3;
//锁存器控制脚__sbit __at (0xB6) wela; //sbit wela=P3^6;
__sbit __at (0xB7) dula; //sbit dula=P3^7;
//定义全局变量uchar a,b;  //两队分数__bit PauseFlag=1;
//暂停标识uchar sec24s=24;
//24s计时uchar sec=0,sec2=0;uchar fen,miao,PauseFen=0,PauseMiao=0;
extern void DisplayTime(void);
extern void DisplayGrade(void);
extern void beep(uchar);
extern void delay(uchar);
extern void ds_init();
extern void init();
extern void keyscan();
extern uchar read_ds(uchar );
extern void write_ds(uchar , uchar );
extern void switchTeam();
extern void teamAdd();
extern void teamSub();
extern void pause();
#endif

/* * =====================================================================
* *       Filename:  main.c
* *    Description:  main source code for jsq
* *        Version:  1.0
*        Created:  27.11.2011 11:52:13
*       Revision:  none
*       Compiler:  sdcc 3.0.0
*         Author:  andyhuzhill, andyhuzhill@gmail.com
*        Company:  http://www.andylinux.co.cc
* ===================================================================== */
#include <8052.h>
#include "main.h"
void delay(unsigned char i)
{
    uchar j;
    while (i--)
        for (j = 0; j < 115; j++);
}
void write_ds(uchar add, uchar ds_data)
{
    dscs = 0;
    dsas = 1;
    dsds = 1;
    dsrw = 1;
    P0 = add;
    dsas = 0;
    //锁存数据
    dsrw = 0;
    P0 = ds_data;
    dsrw = 1;
    dsas = 1;
    dscs = 1;
    //都根据时序图
}
uchar read_ds(uchar add)
{
    uchar  ds_data;
    dsas = 1;
    dsds = 1;
    dsrw = 1;
    dscs = 0;
    P0 = add;
    dsas = 0;
    dsds = 0;
    P0 = 0xff;
    ds_data = P0;
    dsds = 1;
    dsas = 1;
    dscs = 1;
    return ds_data;
}//读写DS12CR887
void ds_init(void)
{
    dsas = 0; //以下未初始化，很重要
    dsds = 0;
    dsrw = 0;
    write_ds(0x0b, 0x04); //DS12cr887使用二进制编码
    write_ds(0x0a, 0x20); //DS12CR887寄存器A功能设置，开启时钟振荡器
}
void keyscan(void)
{
    if (K1 == 0)
    {
        delay(5);
        a++;
        while (!K1);
    }
    if (K2 == 0)
    {
        delay(5);
        a--;
        while (!K2);
    }
    if (K3 == 0)
    {
        delay(5);
        b++;
        while (!K3);
    }
    if (K4 == 0)
    {
        delay(5);
        b--;
        while (!K4);

    }
}
void DisplayTime(void)
{
    wela = 1;
    P2 = sw[3];
    wela = 0;
    dula = 1;
    P2 = szy[(FEN - fen) / 10];
    dula = 0;
    delay(5);
    wela = 1;
    P2 = sw[2];
    wela = 0;
    dula = 1;
    P2 = sz[(FEN - fen) % 10];
    dula = 0;
    delay(5);
    wela = 1;
    P2 = sw[5];
    wela = 0;
    dula = 1;
    P2 = sz[(60 - miao) / 10];
    dula = 0;
    delay(5);
    wela = 1;
    P2 = sw[4];
    wela = 0;
    dula = 1;
    P2 = szy[(60 - miao) % 10];
    dula = 0;
    delay(5);
    //显示小节时间倒计时
    P1_5 = 0;
    dula = 1;
    P2 = szy[sec24s / 10];
    dula = 0;
    delay(5);
    P1_5 = 1;
    P1_4 = 1;
    dula = 1;
    P2 = sz[sec24s % 10];
    dula = 0;
    delay(5);
    P1_4 = 0;
    //24s倒计时显示    wela=1;    P2=0x24;    wela=0;    delay(5);
}

void DisplayGrade(void)
{
    wela = 1;
    P2 = sw[0];
    wela = 0;
    dula = 1;
    P2 = sz[a / 10];
    dula = 0;
    delay(5);
    wela = 1;
    P2 = sw[1];
    wela = 0;
    dula = 1;
    P2 = sz[a % 10];
    dula = 0;
    delay(5);
    //显示A队分数    wela=1;    P2=sw[6];    wela=0;    dula=1;    P2=sz[b/10%10];    dula=0;    delay(5);    wela=1;    P2=sw[7];    wela=0;    dula=1;    P2=sz[b%10];    dula=0;    delay(5);
    //显示B队分数
}

void beep(uchar t)
{
    fmq = 1;
    delay(t);
    fmq = 0;
}

void sec24reset(void) __interrupt 0 //24秒复位函数
{    sec24s = 24;    sec = 0;    TH1 = (65536 - 50000) / 256;    TL1 = (65536 - 50000) % 256;}

void sec24sub(void) __interrupt 3
{
    if (!PauseFlag)
    {
        if (sec == 20)
        {
            sec24s--;
            sec = 0;
        }
        else
        {
            sec++;
        }
        if (sec24s == 0)
        {
            beep(5);
            sec24s = 24;
        }
    }
    else        {   }
}

void pauseint(void) __interrupt 2  //暂停
{
    if (PauseFlag)
    {
        write_ds(0, miao);
        write_ds(2, fen);
        PauseFlag = 0;
    }
    else{   
             fen = read_ds(2);   
             miao = read_ds(0);        
             PauseFlag = 1;    }
}

void pause()
{
    fen = 0;
    miao = 0;
    PauseFlag = 1;
}

void init(void)  // 总初始化函数
{
    EA = 1; //全局中断允许
    ET1 = 1; //T1中断允许
    EX0 = 1;
    EX1 = 1; //外部中断允许
    TR0 = 1;
    TMOD = 0x10;
    //TMOD=00010000B T1工作在模式1
    TH1 = (65536 - 50000) / 256;
    TL1 = (65536 - 50000) % 256;
    TR1 = 1;  //启动定时器T1
    ds_init();
    write_ds(2, 0);
    write_ds(0, 0);
    //将分和秒写入时钟芯片
    a = 0;
    b = 0;
    wela = 0;
    dula = 0;
}

/*****************主函数****************/
void main(void)
{
    init();
    while (1)
    {
        keyscan();
        if (!PauseFlag)
        {
            fen = read_ds(2);
            miao = read_ds(0);
        }
        DisplayGrade();
        DisplayTime();
        if ((fen == 12) && (miao == 0))
        {
            beep(10);
            pause();
        }
    }
}


{% endhighlight %}
