---
layout: post
title: "树莓派上搭建基于Python+web.py+fastcgi+lighttpd的网站"
description: ""
categories: web 
tags: 
- 树莓派
- ARM
---
{% include JB/setup %}




最近在网上淘了一个树莓派，什么是树莓派？这里是他的[官方网站](http://www.raspberrypi.org)你可以去看看。 简单的说就是一块使用了ARM11的CPU，具有256MB或512MB内存的具有两个USB接口，一个RJ45接口，HDMI输出和A/V输出的小开发板。他的特别之处就是所有全部东西都集成在一块银行卡大小的PCB上。官方价格只要$35。    

网上有很多关于如何使用树莓派的创意，比如这个[链接](http://www.shumeidiy.com/thread-29-1-1.html)有34个点子。
 
----

我这里讲的是做一个Web服务器。    

## 安装树莓派系统  

首先，我们要为树莓派安装一个操作系统。官方在[这里](http://www.raspberrypi.org/downloads)提供了四种操作系统可供下载。不过你可以可以在各个树莓派的论坛上找到Android、或其他发行版的linux系统。 我使用的是官方推荐的`Raspbian “wheezy”`，他是基于debian的一个Linux系统。而且他安装软件也比较方便。     

我自己的电脑上运行的就是debain系统。在linux系统下可以很简单的制作树莓派的SD卡。树莓派本身没有带flash，于是只有通过一张SD卡作为“硬盘”了。制作启动SD卡的命令如下：

    dd if=[your-raspberrypi-image.img] of=/dev/[your-sd-card]

if后接的参数是你下载的树莓派的映像文件，of后面接你的SD卡的设备节点。     

制作好之后，这张卡就可以启动树莓派了。将他插入树莓派的SD卡槽中，然后给他上电。我没有显示器，所以我用了一个USB转TTL的线连接到树莓派上，然后用串口调试助手登陆了树莓派。     

## 配置系统

你可以使用默认的root用户，密码是raspberry登陆系统。然后运行 raspi-config 简单配置一下系统。我是用来做Web服务器用的，没有必要使用显示器，所以将GPU的显存设为0。

## 安装程序

接着安装所需要的程序，因为是使用基于debian的系统，所以可以使用有超级牛力的apt-get来安装程序。

    #首先更新缓存并升级软件
     apt-get update && apt-get upgrade

    #然后安装所需的程序
     apt-get lighttpd python-webpy python-flup mysql-server-5.5 mysql-client-5.5
    # lighttpd是网页服务器 python-flup的是为了让python支持fastcgi mysql是数据库  
    # 你也可以使用其他的数据库，不过，我使用过postgresql之后，还是觉得mysql比较习惯

使用以上几个命令就能将所需的软件安装好了，接下来就是配置了。   
 
##配置lighttpd

  lighttpd的配置文件在/etc/lighttpd目录下，主配置文件是lighttpd.conf，这个文件并不需要太多的改动，我的文件内容如下:

    
    server.modules = (
	    "mod_access",
	    "mod_alias",
	    "mod_compress",
 	    "mod_redirect",
        "mod_rewrite",   
	    "mod_status",
    )

    # 下面一行是规定网站的根目录在哪
    server.document-root        = "/var/www"
    server.upload-dirs          = ( "/var/cache/lighttpd/uploads" )
    server.errorlog             = "/var/log/lighttpd/error.log"
    server.pid-file             = "/var/run/lighttpd.pid"
    server.username             = "www-data"
    server.groupname            = "www-data"

    #这里添加一个"index.py"
    index-file.names            = ( "index.py", "index.php", "index.html",
                                "index.htm", "default.htm",
                               " index.lighttpd.html" )

    url.access-deny             = ( "~", ".inc" )

    #这里添加一个 ".py"
    static-file.exclude-extensions = ( ".php", ".pl", ".fcgi" )

    include_shell "/usr/share/lighttpd/use-ipv6.pl"

    dir-listing.encoding        = "utf-8"
    server.dir-listing          = "enable"

    compress.cache-dir          = "/var/cache/lighttpd/compress/"
    compress.filetype           = ( "application/x-javascript", "text/css", "text/html", "text/plain" )

    include_shell "/usr/share/lighttpd/create-mime.assign.pl"
    include_shell "/usr/share/lighttpd/include-conf-enabled.pl"


然后是添加fastcgi支持。直接执行以下命令即可：

    lighty-enable-mod fastcgi

然后修改一下 /etc/lighttpd/conf-enabled/10-fastcgi.conf

    
    # /usr/share/doc/lighttpd-doc/fastcgi.txt.gz
    # http://redmine.lighttpd.net/projects/lighttpd/wiki/Docs:ConfigurationOptions#mod_fastcgi-fastcgi

    server.modules += ( "mod_fastcgi" )

    #添加如下两段
    fastcgi.server = ("index.py" =>
	    ((
	    "socket" => "/tmp/fastcgi.socket",
	    "bin-path" => "/var/www/index.py",
	    "max-procs" => 10,
	    "bin-environment" => (
	    "REAL_SCRIPT_NAME" => ""
	    ),
	    "check-local" => "disable"
	    ))
	)
	
    url.rewrite-once = (
	    "^/favicon$" => "/static/favicon.ico",
	    "^/static/(.*)$" => "/static/$1",
	    "^/(.*)$" => "/index.py/$1",
	    )
	    
以上配置完成之后，你就可以直接写你的webpy程序，然后把他放在/var/www下就可以了，文件名保存为index.py如上配置文件所示。 (注意已经要把配置文件写正确了，否则，你看不到你的网站就不要怪我啦)

## 示例程序

下面我写一个简单的webpy程序作为示范:

{% highlight python  %}

#!/usr/bin/env python
#  *-* coding: utf-8 *-*

import web

urls = (
    "/(.*)", "index",
    )

app = web.application(urls, globals())

class index:
    def GET(self, name):
        return "Hello " + name

if __name__ == "__main__":
    app.run()

{% endhighlight %}
将以上文件保存为index.py放在/var/www目录下，然后重启lighttpd

    service lighttpd restart

在浏览器中输入 http://localhost/andy

你将会看到浏览器中显示：

    Hello andy


OK， 我的介绍就到这里了，其实以上的搭建webpy+lighttpd网站的过程也适用于其他的基于debian的系统，不一定只能在树莓派上使用。如果你有什么问题，欢迎在下面评论。



