---
layout: post
title: "结束了,我的飞思卡尔"
description: ""
category: Competition
tags:
- Freescale
- K60 
---
{% include JB/setup %}





其实早该写这篇blog了,因为比赛已经在23号的时候就结束了。从去年看到比赛规则出来，一时兴起，跑去和沈春阳说我也想做飞思卡尔这个比赛，到现在也算是大半年了，实际准备的时间可能也就两三个月。
    
## 开始阶段
一开始，对于这个比赛也不是很了解，只知道沈春阳参加过，而且觉得自己做出一辆小车，让他在赛道上飞驰，是件很有意思的事情。我因为之前每次比赛都是写程序的，于是第一件事情就是熟悉飞思卡尔比赛规定的芯片。比赛规定主控芯片必须使用飞思卡尔公司生产的芯片，共有六个系列：

    1. 32位Kinetis系列
    2. 32位ColdFire系列
    3. 32位MPC56xx系列
    4. 16位9S12系列
    5. DSC系列
    6. 8位单片机系列（可使用2片）

Kinetis系列是飞思卡尔公司最新推出的基于ARM-Cortex-M4内核的单片机，正好我也学过一些ARM的知识，于是就成了我们选择的芯片。其实其他系列也算历史悠久，ColdFire系列据说是原Motorola公司生产的M68K系列基础上开发的，第一台苹果电脑使用的就是M68K系列的CPU。 MPC56xx系列则是使用的著名的PowerPC的指令集，许多的超级计算机就是使用的PowerPC指令集的CPU。 


一开始，对于K60芯片，我也是无处下手，只有看网上各种资料和数据手册。STM32的芯片，官方有提供一个库，直接调用他的库就能使用他的芯片了，而飞思卡尔没有提供一个类似的库给K60,这也是两家公司的一个设计思想不同吧，其实飞思卡尔在他自己的开发工具CodeWarror里面提供了一个Processor Expert 工具，可以通过设置后，自动生成一些代码，然后就直接调用生成的代码就可以了。只是很多人可能直接就用的IAR、Keil之类的开发工具，就不知道有这么一个东西了吧。
    
网上有很多看准飞思卡尔这个比赛商机的商家，他们提供了各种各样的K60核心板，也提供了他们自己写的底层库，方便客户使用。我就使用了野火提供的K60底层库，将它移植到了CodeWarrior开发环境中。
   
直到大三上学期结束，我也就是在干着熟悉芯片，然后移植别人的底层库的工作。
我的队友也在这个时候，用实验室的制版机，做了我们的第一块电路板，这块电路板也只是一个电机驱动的测试电路。
    ![](/images/freescale/1.jpg)

这个时候，我们只是刚刚组装好了车模，买来了野火的鹰眼ov7725摄像头，然后就各自回家了。


## 进行阶段

第二个学期开始，我们就有些紧张了，在网上看到各种视频，各个论坛上有人发帖说车子已经跑到多少米每秒了。

我们第一块电路板也发出去做了回来。
    ![](/images/freescale/2.jpg)
结果就是两个巨大的问题。
    (一） 摄像头的线序刚好和插座上的线序是左右相反的，摄像头插上去没法用。
   （二） 控制电路与驱动电路之间用的光耦隔离是有问题的，光耦出来的波形几乎都是高电平。


于是又改电路板：

![](/images/freescale/3a.jpg)
![](/images/freescale/3b.jpg)

这次电路板暂时能用了，但是还是没法跑起来,我照着网上搜刮来的程序，自己大概写了一个程序,不过，只要舵机一动，单片机就重启了！这还了得，程序没法运行啊。赶紧查问题，结果发现是舵机的供电问题，没有独立供电，舵机一动，单片机的电压就迅速的被拉低了，自然就重启了。这会，我们先用单独的DC-DC降压模块先给舵机供电，让它跑了起来。然后叫做硬件的曹亮同学做了好几个电源模块，一开始的线性稳压芯片没接舵机，能正常稳压到5V，结果舵机一动，电压又被拉低了，最后只好用了DC-DC降压模块。

下面这个视频是使用这块电路板，然后外加的独立的DC-DC模块给舵机供电的。
 <embed src="http://player.youku.com/player.php/sid/XNTg4ODE3NDA4/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>

于是改了电路板，将舵机供电独立用了DC-DC模块。

![](/images/freescale/4a.jpg)
![](/images/freescale/4b.jpg)

这块板子也不是那么完美无缺，这次出现了一个之前都没有的问题。舵机的定时器和电机的定时器用了同一个定时器！本来K60是有三个FTM定时器，每个定时器可以输出8路不同占空比的信号，但是同一个FTM定时器只能使用一个频率，而舵机控制和电机控制是需要不同的频率的。两种解决方法：软件上，两个都使用同一个频率，这个当然不行。还有一个就是改变占空比的函数改成整个初始化，这个也有一个问题，就是舵机和电机的控制相当于两个频率不停的变换。所以软件上的改变没有什么好处。最后只有在硬件上改动了，将原来舵机控制的线割断，然后飞了一根线过去。

这块电路板一直用到了最后参加华南赛区比赛。


##  比赛和准备比赛

当改完第三块电路板的时候，马上就是湖南省的省赛了。这个节骨眼上，又出问题了，车子的齿轮被磨坏了！ 这种问题居然都会被我碰上。 调试那天，车子没有齿轮，我向其他组没有来参加省赛的借齿轮，不过没有这么快能到，于是，我只好用手推着小车在调试跑道上跑跑了。

第二天，正式比赛，我换上了借来的齿轮。程序也没有怎么调，最后用一个很慢的速度，跑完了初赛的两轮，两轮成绩都是一样——27s，因为没有在起跑线停住还加了一秒。

省赛可以说是正式见识到了这个比赛到底是怎么一个流程，也知道要准备一些什么了。

离华南赛区的比赛还有20多天，还可以提升一下速度。速度一上来，之前的一些没出现的问题也暴露出来了，我们这边铺的跑道，由于被人踩，被另外一组的车砸，已经坑坑洼洼，而且很多灰，我们摄像头组的车轮刚好很怕脏，只要有一点点灰尘，就容易打滑，在弯道根本没法提速。这时我听说城南学院那边用省赛时的跑道铺了个新的跑道，我就去蹭他们的跑道:-P，他们的跑道维护的比我们好，干净很多。下面就是我的车在那里跑的视频：
<embed src="http://player.youku.com/player.php/sid/XNTg2NzUyOTIw/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>

最后，我们终于启程，到了武汉。一下车就感觉武汉真心热。然后坐了一个多小时的车到了华中科技大学光谷体育馆
![](/images/freescale/guanggu.jpg)
![](/images/freescale/huake.jpg)

华科的树真心多啊！校园感觉就是在一个树林里面建的。

第一天报道，第二天试车，第三天预赛，第四天决赛。

比赛结束，我们组获得了摄像头组二等奖。比赛中也还是有一些些的遗憾，不过也有更高兴的事情，那就是认识了一些一起玩车的人，他们和我们一样为了同一个目标，更快、更强努力过，同样也遇到过我们所遇到过的一些问题，当我们聊起来的时候，就觉得特别亲切。

回来的时候，我还在武昌火车站碰到了一个网友，之前一直只在网上交流，这回真正的面对面了，还是有些巧合的，本来他们的车应该18点多就开的，结果晚点了四个多小时，刚好被我碰到了，哈哈。


最后附上一张我与厦大发车妹纸的合照，她是我初中同学，呵呵。

![](/images/freescale/wanglei.png)
