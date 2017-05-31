---
layout: post
title: "LaTeX Test"
description: ""
categories: 
tags: []
---
{% include JB/setup %}
* TOC
{:toc}
<hr/>

if you can see a equation 

$$
e^x = \sum_{n=0}^\infty \frac{x^n}{n!} = \lim_{n\rightarrow\infty} (1+\frac{x}{n})^n 
$$



{% highlight c %}
#include <stdio.h>
int
main(int argc, char *argv[])
{
    printf("hello world!\n");
    return 0;
}
{% endhighlight %}

{% highlight pascal %}
program hello;
begin
    writeln('hello world');
end.
{% endhighlight %}

{% highlight bash %}
#!/usr/bin/bash

for $i in (100, 100)
    do 
        ping 11.1.1.$i
    done
{% endhighlight %}
