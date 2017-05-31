---
comments: true
date: 2012-01-24 14:26:00
layout: post
title: GNU編碼標準中關於使用內存分配例程的論述
categories:
- Linux
- ProgramminginC
tags:
- C
---

{% include JB/setup %}
* TOC
{:toc}
<hr/>
每次調用malloc或realloc時都要進行檢查，看他們是否返回零。即使縮小內存塊，也要檢查realloc是否返回零。有些系統把內存塊大小圓整爲2的整數次冪，在這些系統上，如果你要求分配更少的空間，那麼realloc可能得到一個不同的內存塊。

在Unix環境下，如果realloc返回零，那麼它會清除原來的內存塊。GNU realloc沒有這個Bug: 如果函數分配失敗，原始塊不會有變化，你只管假設這個bug已經修復了。如果你希望在Unix環境下運行你的程序，並且希望避免分配失敗情況下丟失原始塊，那麼你可以使用GNU malloc。  


你必須明白，free會改變那些釋放掉的內存塊的內容。因而，你必須在調用free之前，從內存塊中提取出所有你想提取的東西。  

