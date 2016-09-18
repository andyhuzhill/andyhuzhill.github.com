---
comments: true
date: 2012-07-19 00:55:57
layout: post
title: 串口编程 使用Qt 或 Lazarus
categories:
- Linux
- pascal
- Qt
tags:
- lazarus
- Linux
- Qt
- 串口编程
- 编程
---

{% include JB/setup %}
* TOC
{:toc}
<div style="border-bottom: 1px solid #ccc;line-height: 1.3em;"></div>
最近几天，我室友在学习Gtk+，然后用Gtk+在 Linux下实现了一个串口调试助手,他的调试助手在这里有介绍[http://yeziquan.sinaapp.com/qrs/OS/APP/gui/gtk_serial.html用gtk写一个linux下的串口调试助手](http://yeziquan.sinaapp.com/qrs/OS/APP/gui/gtk_serial.html)。我也写一个姊妹篇。

今天我就用几个GUI编程工具编写串口程序的方法，做一下介绍。至于串口是什么，以及串口编程的一些基本知识，就不是我这篇文章的重点了。



### 串口编程 Qt篇


Qt中并没有直接提供串口编程所需要的类库。不过你也可以直接调用所使用系统的API编程。以下是一个Linux系统下打开串口的示例函数：

{% highlight cpp  %}
    
    int TMainForm::openSerialPort()
    {
    int fd = -1;
    const char *devName = "/dev/ttySAC2";
    fd = ::open(devName, O_RDWR|O_NONBLOCK);
    if (fd < 0) {
    return -1;
    }
    termios serialAttr;
    memset(&serialAttr;, 0, sizeof serialAttr);
    serialAttr.c_iflag = IGNPAR;
    serialAttr.c_cflag = B115200 | HUPCL | CS8 | CREAD | CLOCAL;
    serialAttr.c_cc[VMIN] = 1;
    if (tcsetattr(fd, TCSANOW, &serialAttr;) != 0) {
    return -1;
    }
    return fd;
    }
    
{% endhighlight %}


这样如果你如果熟悉所使用系统的API可以直接写，不过不具有可移植性。
下面介绍一个第三方开发的可移植的串口编程类库——Qextserialport。
它的主页是[http://code.google.com/p/qextserialport/](http://code.google.com/p/qextserialport/)
这个类库的源文件可以通过如下的git命令下载：

    
    
    git clone https://code.google.com/p/qextserialport/
    
    或者是
    
    git clone http://code.google.com/p/qextserialport/
    


文件中的examples文件夹中有几个示例，你可以通过阅读代码学习如何使用这个类库。
使用的时候将src文件下的几个文件都复制到你自己的代码目录下，然后将他们添加进工程。
主要是

    qextserialport.cpp 
    qextserialport.h
    qextserialport_global.h
    qextserialport_p.h  // posix 系统下使用的头文件定义
    qextserialport_unix.h // unix系统下使用的头文件定义
    qextserialport_win.h  //windows系统下使用的头文件定义

使用Qt写的串口调试助手（linux版） 源代码可以在这里下载到
[Mycom](http://andyhuzhill.github.com/MyQtCom)

这个串口调试助手只是简单的实现的发送和接收字符串。

写程序时注意一下打开串口的串口名，windows下一般是COM1、COM2 ，Linux下一般是 /dev/ttyS1、/dev/ttyUSB0。
还有一点是windows下的串口打开方式有Polling（轮询）和EventDriven（事件驱动）两种模式，而Linux则只有Polling（轮询模式）。
记住这两点，基本上就能写出Windows上、linux上都可以编译执行的程序了。


### 串口编程  Lazarus篇

Lazarus是一个基于Free Pascal的类似于Delphi的RAD（Rapid Application Development)工具。
使用Lazarus写串口程序，也有好几种方式，首先是和Qt的第一种方式一样，直接调用系统提供的API函数。特别是Windows和WinCE平台上，有人专门为串口编程写了一个类。类文件的内容如下

    
{% highlight pascal  %}

    unit CE_Series;
    interface
    uses
    Windows,Classes, SysUTIls, LResources, StdCtrls,ExtCtrls;
    type
    TCE_Series = class(TObject)
    private
    hComm: THandle;
    public
    Function Openport(Port:LPCSTR;BaudRate,ByteSize,Parity,StopBits:integer):String;
    procedure Send(str:String);
    Function Receive():String;
    procedure closePort();
    end;
    implementation
    //===============================================================================================
    // 语法格式:OpenPort(Port:LPCWSTR;BaudRate,ByteSize,Parity,StopBits:integer)
    // 实现功能:打开串口
    // 参数:port,串口号;例如wince下为从COM1:,COM2:……。win32下为COM1,COM2.…… ;其他略,顾名思义哈
    // 返回值:错误信息
    //===============================================================================================
    function TCE_Series.OpenPort(Port:LPCSTR;BaudRate,ByteSize,Parity,StopBits:integer):String;
    var
    cc:TCOMMCONFIG;
    begin
    result:='';
    hComm:=CreateFile(port, GENERIC_READ or GENERIC_WRITE,
    0, nil, OPEN_EXISTING, 0, 0); // 打开COM
    if (hComm = INVALID_HANDLE_VALUE) then begin // 如果COM 未打开
    result:='CreateFile Error!';
    exit;
    end;
    GetCommState(hComm,cc.dcb); // 得知目前COM 的状态
    cc.dcb.BaudRate:=BaudRate; // 设置波特率为BaudRate
    cc.dcb.ByteSize:=ByteSize; // 字节为 ByteSize(8 bit)
    cc.dcb.Parity:=Parity; // Parity 为 None
    cc.dcb.StopBits:=StopBits; // 1 个Stop bit
    if not SetCommState(hComm, cc.dcb) then begin// 设置COM 的状态
    result:='SetCommState Error!';
    CloseHandle(hComm);
    exit;
    end;
    end;
    //===============================================================================================
    // 语法格式:Send(str:String)
    // 实现功能:发送数据
    // 参数:str,数据
    // 返回值:无
    //===============================================================================================
    procedure TCE_Series.Send(str:String);
    var
    lrc:LongWord;
    begin
    if (hComm=0) then exit; //检查Handle值
    WriteFile(hComm,str,Length(str), lrc, nil); // 送出数据
    end;
    //=====================================================================
    //语法格式: Receive()
    //实现功能: 接收串口数据
    //参数: 无
    //返回值: 收到的字符串
    //=====================================================================
    Function TCE_Series.Receive():String;
    var
    inbuff: array [0..2047] of Char;
    nBytesRead, dwError:LongWORD ;
    cs:TCOMSTAT;
    begin
    ClearCommError(hComm,dwError,@CS); //取得状态
    // 数据是否大于我们所准备的Buffer
    if cs.cbInQue > sizeof(inbuff) then begin
    PurgeComm(hComm, PURGE_RXCLEAR); // 清除COM 数据
    exit;
    end;
    ReadFile(hComm, inbuff,cs.cbInQue,nBytesRead,nil); // 接收COM 的数据
    //转移数据到变量中
    result:=Copy(inbuff,1,cs.cbInQue);//返回数据
    end;
    //=====================================================================
    //语法格式: ClosePort()
    //实现功能:关闭串口
    //参数: 无
    //返回值: 无
    //=====================================================================
    procedure TCE_Series.ClosePort();
    begin
    SetCommMask(hcomm,$0);
    CloseHandle(hComm);
    end;
    end.
    

{% endhighlight %}

这个类很简单，提供了四个函数，打开串口，读、写串口，关闭串口。将以上内容直接保存为一个名为ce_series.pas的文件，复制到你的代码的文件夹。
使用示例如下：

{% highlight pascal  %}
    
    unit Unit1;
    {$mode objfpc}{$H+}
    interface
    uses
      Classes, SysUtils, FileUtil, Forms, Controls, Graphics, Dialogs, StdCtrls,
      ExtCtrls, CE_Series; //末尾添加类文件
    type
    
      { TForm1 }
    
      TForm1 = class(TForm)
        Button1: TButton;
        procedure Button1Click(Sender: TObject);
        
      private
        { private declarations }
      public
        { public declarations }
      end; 
    
    var
      Form1: TForm1;
      mySerial:TCE_Series; //申明一个对象mySerial
     
    implementation
    
    
    procedure TForm1.Button1Click(Sender: TObject);
    var
      error:string;
    begin
      mySerial:= TCE_Series.Create;  // 实例化对象mySerial
      error:=mySerial.Openport('COM4',9600,8,0,1); //打开串口，windows下串口名为COM4，WinCE下串口名则为COM4:
      if (error ='CreateFile Error!')  or (error= 'SetCommState Error!') then
      begin
         ShowMessage('串口打开错误');
         exit;
      end;
    end;
    
{% endhighlight %}

以上的CE_Series类实际上是调用了Windows的CreateFile这个API，所以只适用于Win32和WinCE。

为了写出更具可移植性的程序，并且使用更多的串口的功能，Lazarus可以使用如下介绍的一个类库——SdpoSerial；
这个类库可以在这里下载到
[http://sourceforge.net/projects/sdpo-cl/](http://sourceforge.net/projects/sdpo-cl/)
下载之后使用Lazarus安装里面的sdposeriallaz.lpk包即可。安装完毕，会在lazarus的控件列表里面发现一个Sdpo的标签页，只有一个如下的图标[![](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/07/Screenshot-3.png)](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/07/Screenshot-3.png)
将它拖到你的程序的窗体上，就可以使用这个类了，这个串口控件是一个不可见控件。
[![](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/07/Screenshot-4-1024x537.png)](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/07/Screenshot-4.png)
程序示例如下

{% highlight pascal  %}    
    
    procedure TForm1.Button1Click(Sender: TObject);
    var
    str:string;
    begin
      SdpoSerial1.Device:='/dev/ttyS0';  //使用/dev/ttyS0串口
      SdpoSerial1.BaudRate:=br__9600;    //波特率设定为9600
      SdpoSerial1.Parity:=pOdd;          //校验位设为奇校验
      SdpoSerial1.StopBits:=sbOne;       //停止位设为1位
      SdpoSerial1.Active:=true;          //启动串口
      SdpoSerial1.Open;                  //打开串口
      str:=SdpoSerial1.ReadData;         //读取串口数据
      SdpoSerial1.WriteData('hello world');  // 写入串口数据
      SdpoSerial1.Close;                  //关闭串口
    end; 
    
{% endhighlight %}

具体的这个类提供了哪些函数、过程和属性，可以直接查看他的源文件SdpoSerial.pas。
注意：SdpoSerial类库的说明文件里貌似说无法使用在WinCE系统上，我没有测试过，各位可以自己测试一下。



### 小结


基本上，各种编程工具都会提供直接访问系统API的方式来进行串口编程，有些人还会写出封装好了的串口编程的类库可供我们使用。使用这些类库，我们有时候不必考虑各个系统上串口编程方式的不同，从而方便的实现程序的可移植性。

