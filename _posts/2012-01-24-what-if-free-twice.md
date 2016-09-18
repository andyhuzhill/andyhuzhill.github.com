---
comments: true
date: 2012-01-24 13:55:00
layout: post
title: Free一個動態指針兩次會怎樣？
categories:
- ProgramminginC
tags:
- C
---

{% include JB/setup %}
* TOC
{:toc}
<div style="border-bottom: 1px solid #ccc;line-height: 1.3em;"></div>
教科書上說 兩次釋放相同的指針會導致“不確定的行爲”，那麼 我就試一試寫了下面一個簡單代碼

    #include <stdio.h>

    #include <stdlib.h>

    int main()

    {

    char *p;

    p=(char *)malloc(sizeof(char));

    free(p);

    free(p);

    return 0;

    }   

  


gcc -o freetwice freetwice.c

    ./freetwice

    *** glibc detected *** ./freetwice: double free or corruption (fasttop): 0x09537008 ***

    ======= Backtrace: =========

    /lib/i386-linux-gnu/libc.so.6(+0x6ebc2)[0x17ebc2]

    /lib/i386-linux-gnu/libc.so.6(+0x6f862)[0x17f862]

    /lib/i386-linux-gnu/libc.so.6(cfree+0x6d)[0x18294d]

    ./freetwice[0x8048445]

    /lib/i386-linux-gnu/libc.so.6(__libc_start_main+0xf3)[0x129113]

    ./freetwice[0x8048381]

    ======= Memory map: ========

    00110000-00286000 r-xp 00000000 08:09 655379  /lib/i386-linux-gnu/libc-2.13.so

    00286000-00288000 r--p 00176000 08:09 655379  /lib/i386-linux-gnu/libc-2.13.so

    00288000-00289000 rw-p 00178000 08:09 655379  /lib/i386-linux-gnu/libc-2.13.so

    00289000-0028c000 rw-p 00000000 00:00 0

    00808000-00824000 r-xp 00000000 08:09 655404  /lib/i386-linux-gnu/libgcc_s.so.1

    00824000-00825000 r--p 0001b000 08:09 655404  /lib/i386-linux-gnu/libgcc_s.so.1

    00825000-00826000 rw-p 0001c000 08:09 655404  /lib/i386-linux-gnu/libgcc_s.so.1

    008b7000-008b8000 r-xp 00000000 00:00 0  [vdso]
    00dc7000-00de5000 r-xp 00000000 08:09 655376  /lib/i386-linux-gnu/ld-2.13.so

    00de5000-00de6000 r--p 0001d000 08:09 655376  /lib/i386-linux-gnu/ld-2.13.so

    00de6000-00de7000 rw-p 0001e000 08:09 655376  /lib/i386-linux-gnu/ld-2.13.so

    08048000-08049000 r-xp 00000000 08:07 3009831  /home/lenovo/Myprogram/Linux/mem/freetwice

    08049000-0804a000 r--p 00000000 08:07 3009831  /home/lenovo/Myprogram/Linux/mem/freetwice

    0804a000-0804b000 rw-p 00001000 08:07 3009831  /home/lenovo/Myprogram/Linux/mem/freetwice

    09537000-09558000 rw-p 00000000 00:00 0  [heap]

    b7600000-b7621000 rw-p 00000000 00:00 0

    b7621000-b7700000 ---p 00000000 00:00 0

    b7742000-b7743000 rw-p 00000000 00:00 0

    b7758000-b775a000 rw-p 00000000 00:00 0

    bff27000-bff48000 rw-p 00000000 00:00 0  [stack]

    已放弃

  


哈哈 結果就是這樣

 
