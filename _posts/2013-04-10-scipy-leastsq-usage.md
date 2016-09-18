---
layout: post
title: "scipy中最小二乘法函数leastsq的用法"
description: ""
categories: 
- python
- scipy
tags: 
- python
- 最小二乘法
---
{% include JB/setup %}
* TOC
{:toc}
<div style="border-bottom: 1px solid #ccc;line-height: 1.3em;"></div>

好久没有写Blog了，最近都没有啥好写的。 今天我研究了一下scipy里面的那个最小二乘法的函数的用法，一开始，没弄懂那个函数是怎么调用了，只知道敲进示例程序能用，自己写的程序却报错，后来搜索了一下，看了看别人的代码，搞明白了一点。

## 基本原理

首先介绍一下最小二乘法的一个原理吧。最小二乘法是一种数学优化方法，通过最小化误差的平方和寻找最佳的匹配函数。一般常用于曲线的拟合，我最早接触到最小二乘法也是在高中的数学课上用来拟合一次函数曲线。

最小二乘法的基本公式如下: (来自[百度百科][baidubaike])

$$ \frac{\partial\phi}{\partial a_0} = -2 \sum{(Y_i - a_0 - a_iX_i)}  =0 $$ 

$$ \frac{\partial\phi}{\partial a_i} = -2 X_i\sum{(Y_i-a_0-a_iX_i)} = 0 $$

## 简单的一次函数拟合

假设一组数据的分布呈线性，我们就可以用一个函数$y=k\times x +b$ 去拟合它，那么那两个系数 k和b怎么求呢？

直接给出公式解: (参考[维基百科][wiki])


![](http://upload.wikimedia.org/math/0/c/8/0c801c7273d9dbf67f98db26e31063cd.png)
和
$$ x_0 = \bar{y} - x_1\bar{t} $$
 
其中$$\bar{t} = \frac{1}{n} \sum_{i=1}^{n}t_i $$为t值的算术平均值，也可以解得如下形式:
 
$$x_1 = \frac{\sum_{i=1}^n (t_i - \bar t)(y_i - \bar y)}{\sum_{i=1}^n (t_i - \bar t)^2}$$
 
###示例程序
 我用python根据以上算法写了一个简单的一次函数最小二乘法拟合的程序:

{% highlight python  %}
#!/usr/bin/env python
# *-* encoding: utf-8 *-*
#
# =============================================
#      Author   : Andy Scout
#    Homepage   : http://andyhuzhill.github.com
#    E-mail     : andyhuzhill@gmail.com
#
#  Description  :
#  Revision     :
#
# =============================================

from __future__ import division

def leastsq(x,y):
    """
    x,y分别是要拟合的数据的自变量列表和因变量列表
    """
    meanx = sum(x) / len(x)   #求x的平均值
    meany = sum(y) / len(y)   #求y的平均值

    xsum = 0.0
    ysum = 0.0

    for i in range(len(x)):
        xsum += (x[i] - meanx)*(y[i]-meany)
        ysum += (x[i] - meanx)**2

    k = xsum/ysum
    b = meany - k*meanx

    return k,b   #返回拟合的两个参数值

{% endhighlight %}


##使用scipy提供的最小二乘法函数

以上的方法理解起来也比较容易，不过如果需要拟合的函数不是一次函数，就比较麻烦了。python的科学计算包scipy的里面提供了一个函数，可以求出任意的想要拟合的函数的参数。那就是scipy.optimize包里面的leastsq函数。
函数原型是:

    leastsq(func, x0, args=(), Dfun=None, full_output=0, col_deriv=0, ftol=1.49012e-08, xtol=1.49012e-08, gtol=0.0, maxfev=0, epsfcn=0.0, factor=100, diag=None, warning=True)

一般我们只要指定前三个参数就可以了。

func  是我们自己定义的一个计算误差的函数，

x0    是计算的初始参数值

args  是指定func的其他参数

下面用一个示例程序来解释他的用法,同样也是使用上面的求一次函数的拟合参数

{% highlight python  %}
#!/usr/bin/env python
# *-* encoding: utf-8 *-*
#
# =============================================
#      Author   : Andy Scout
#    Homepage   : http://andyhuzhill.github.com
#    E-mail     : andyhuzhill@gmail.com
#
#  Description  :
#  Revision     :
#
# =============================================

import numpy as np
from scipy.optimize import leastsq

from data import x,y  #从外部导入想要拟合的数据, x和y都是list类型

def fun(p, x):
    """
    定义想要拟合的函数
    """
    k, b = p  #从参数p获得拟合的参数
    return k*x+b

def err(p, x, y):
    """
    定义误差函数
    """
    return fun(p,x) -y

#定义起始的参数 即从 y = 1*x+1 开始，其实这个值可以随便设，只不过会影响到找到最优解的时间
p0 = [1,1]   

#将list类型转换为 numpy.ndarray 类型，最初我直接使用
#list 类型,结果 leastsq函数报错，后来在别的blog上看到了，原来要将类型转
#换为numpy的类型

x1 = np.array(x)  
y1 = np.array(y)

xishu = leastsq(err, p0, args=(x1,y1))

print xishu[0]

# xishu[0]，即为获得的参数

{% endhighlight %}

## 后记

看到这里，你应该也能很好的使用scipy提供的leastsq函数做最小二乘法的拟合了吧？
如果你遇到什么问题，或者是我的文中有什么错误，欢迎在文后的评论中指出，谢谢！


[baidubaike]:   http://baike.baidu.com/view/139822.htm#1
[wiki]:  http://zh.wikipedia.org/wiki/%E6%9C%80%E5%B0%8F%E4%BA%8C%E4%B9%98%E6%B3%95
