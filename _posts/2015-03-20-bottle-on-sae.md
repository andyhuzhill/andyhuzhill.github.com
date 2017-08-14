---
layout: post
title: "在SAE上使用Bottle搭建网站"
description: ""
category: 
- python
- bottle.py
tags:
- python
- SAE
---
{% include JB/setup %}





最近在新浪云平台[SAE](http://sae.sina.com.cn)上做了一个网站，这次使用了[Bottle](http://www.bottlepy.org)这一个极其精简的Python 
Web框架。要说Bottle有多简单，它仅仅一个文件3700多行代码就实现了基本的Web框架，还包含了一个简单的模板
系统。如果你兴趣和时间，完全可以读一下它的代码，了解它背后的原理。

## 系统使用的组件

1. bottle.py : Web框架
2. Jinja2    : 模板系统
3. sqlachemy : ORM组件
4. MySQLdb   : MySQl连接驱动
5. beaker    : Sessoins组件

前四个组件在SAE上都已经有预装了，而最后一个是第三方组件，可以使用[SAE]提供的本地开发环境 saecloud 命令下载安装, 也可以从组件的官方网站上下载解压到应用的site-packages目录下。
然后在 index.wsgi 中添加如下代码：

{% highlight python %}

import os
import sys 

app_root = os.path.dirname(__file__)  
sys.path.insert(0, os.path.join(app_root, 'site-packages'))  

{% endhighlight %}


具体的应用程序开发，可以参考Bottle的API文档，看一遍照着做即可。 

## 系统开发中遇到的一些问题以及解决方法

###  MySQL连接出现 ("MySQL gone away")错误

 该问题在SAE的FAQ上已经有了[解答](http://sae.sina.com.cn/doc/python/faq.html#mysql-gone-away)

    MySQL gone away问题¶
    MySQL连接超时时间为30s，不是默认的8小时，所以你需要在代码中检查是否超时，是否需要重连。
    
    对于使用sqlalchemy的用户，需要在请求处理结束时调用 db.session.close() ，关闭当前session，将mysql连接还给连接池，并且将连接池的连接recyle时间设的小一点（推荐为10s）。

###  Sessions 

HTTP协议是一种无状态的协议，也就是说每次连接服务器都无法知道之前的连接有什么信息。
为了记录登陆状态，网站程序会在向用户端发送Cookie，浏览器会保存下Cookie，下次请求服务器的时候会把这个Cookie也发送给服务端。在服务端还会保存一个对应的Session对象来记录用户之前的操作信心。

因为Bottle很简单，所以它也没有实现Session, 我这里使用的是第三方组件Beaker, 不过由于SAE的几个限制，我
直接在网上找到的Beaker的设置选项没法使用。
SAE上没有办法直接读取文件，所以没有办法将Session类型设置为file。
后来查阅Beaker的文档，可以使用数据库保存Beaker的Session，可惜上一个问题有解释，SAE上的MySQL有连接超时
限制，超过30s后，基本上Session就会出错需要重新登陆了。
继续查看Beaker文档，发现Beaker还可以使用memcached作为后端保存Session。于是最终的Beaker配置参数如下:

{% highlight python %}
session_opts = {                                                                                     
    "session.type": "ext:memcached",                                                                 
    "session.url": "127.0.0.1:11211",                                                                
    "session.cookie_expires": True,                                                                  
    "session.timeout": 600,                                                                          
    "session.auto": True                                                                             
    }    
{% endhighlight %}

SAE上的memcached并不需要 session.url参数，不过如果不提供该参数 Beaker会报错，因此随便提供了一个地址。

