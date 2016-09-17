---
comments: true
date: 2012-01-18 23:38:00
layout: post
title: Ubuntu下 NFS的配置
categories:
- Linux
tags:
- Linux
- Ubuntu
---

{% include JB/setup %}
###   嵌入式开发中，如果使用NFS启动开发板上的系统，可以大大减少Flash的擦写次数，提高开发效率，本文就介绍了一下怎么在主机是Ubuntu系统的情况下配置NFS


主机部分

  


一  主机安装NFS服务端

    sudo apt-get install nfs-kernel-server nfs-common

  


二  查看系统有没有nfs功能

    cat /proc/filesystems |grep nfs

三  配置/etc/exports

打开 /etc/exports

末尾加入

    [path_to_share_directory]  [host] [option]

  


  


host是你要向哪个范围内的主机开共享 ×是向所有网段内的主机

option的意义

    ◆ rw 可擦写的权限。   
    ◆ ro 只读的权限。   
    ◆ no_root_squash 
    当登入NFS主机使用共享,被转换成为匿名使用者，通常它的UID与GID都会变  
之目录的使用者如果是root时，那么这个使用者的权限将  
成nobody身份。  
    ◆ root_squash 
    登入NFS主机就具有 root的权限。  
机使用共享目录的使用者，如果是ro  
ot，那么对于这个共享的目录来说，它  
　　◆ all_squash 不论登入使用者的身份为何，它的身份都会被转换成为匿名使用者，通常也就是  
　　◆ anonuid 通常为nobody，当然也可以自行  
设定这个UID的值，UID必须存在于/etc/passwd当中。  
　　◆ anongid 同anonuid，但是变成group ID就是了。   
　　◆ sync 资料同步写入到内存与硬盘当中。   
　　◆ async 资料会先暂存于内存当中，而非直接写入硬盘。    


  


  


man手册里的 exports文件实例

    # sample /etc/exports file

    /  master(rw) trusty(rw,no_root_squash)

    /projects  proj*.local.domain(rw)

    /usr  *.local.domain(ro) @trusted(rw)

    /home/joe  pc001(rw,all_squash,anonuid=150,anongid=100)

    /pub  *(ro,insecure,all_squash)

    /srv/www  -sync,rw server @trusted @external(ro)

    /foo  2001:db8:9:e54::/64(rw) 192.0.2.0/24(rw)

    /build  buildhost[0-9].local.domain(rw)

  


  


四 执行更新设置

    sudo exportfs -a   


五  重启服务

    #sudo /etc/init.d/portmap start  
    #sudo /etc/init.d/nfs-kernel-server start  


  


六 查看所有共享

    showmount -e

七 测试共享

    # mount -t nfs localhost:/opt/FriendlyARM/mini2440/root_qtopia /mnt/

  


如果没有出错信息，在mnt下就应该可以看到和/opt/FriendlyARM/mini2440/root_qtopia一样的内容了。  


也可以在开发板的终端输入以下命令:(假设主机的ip地址为192.168.1.139)

    #mount –t nfs –o nolock 192.168.1.139:/opt/FriendlyARM/mini2440/root_qtopia /mnt

挂接成功后，在开发板的/mnt下就应该可以看到和主机/opt/FriendlyARM/mini2440/root_qtopia一样的内容了。

取消挂接的命令如下：

    #umount /mnt

使用restart和stop可以重启和停止nfs服务，回到主机，在主机终端里输入

    #sudo /etc/init.d/nfs-kernel-server restart

    #sudo /etc/init.d/nfs-kernel-server stop

  


 
