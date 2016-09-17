---
layout: post
title: "CUDA 编程模型"
description: ""
category: "Parallel Programming"
tags: [cuda, parallel programming]
---
{% include JB/setup %}


怎么样才能让代码在GPU上运行呢？ CUDA已经提供了一个很方便的编程环境，编程者只需要掌握了标准的C语言编程，再了解一下GPU编程的特点就可以写出能在GPU上运行的程序了，这里我用一个很简单的程序作为例子。

## 第一个CUDA程序 
 
{% highlight cuda %}
#include <stdio.h>

#define ITEM_COUNT 100

__global__ void add(int *a, int *b, int *c)
{
	int idx = threadIdx.x;
	c[idx] = a[idx] + b[idx];
}

int main(int argc, char **argv)
{
	int h_array_a[ITEM_COUNT] = {0};
	int h_array_b[ITEM_COUNT] = {0};
	int h_array_c[ITEM_COUNT] = {0};

	int *d_array_a = NULL;
	int *d_array_b = NULL;
	int *d_array_c = NULL;

	for (int i = 0; i < ITEM_COUNT; ++i) {
		h_array_a[i] = i;
		h_array_b[i] = i;
	}
	memset(h_array_c, 0, ITEM_COUNT * sizeof(int));

	cudaMalloc(&d_array_a, ITEM_COUNT * sizeof(int));
	cudaMalloc(&d_array_b, ITEM_COUNT * sizeof(int));
	cudaMalloc(&d_array_c, ITEM_COUNT * sizeof(int));

	cudaMemcpy(d_array_a, h_array_a, ITEM_COUNT * sizeof(int), cudaMemcpyHostToDevice);
	cudaMemcpy(d_array_b, h_array_b, ITEM_COUNT * sizeof(int), cudaMemcpyHostToDevice);
	cudaMemset(d_array_c, 0, ITEM_COUNT * sizeof(int));

	add<<<1, ITEM_COUNT>>>(d_array_a, d_array_b, d_array_c);

	cudaThreadSynchronize();
	cudaMemcpy(h_array_c, d_array_c, ITEM_COUNT * sizeof(int), cudaMemcpyDeviceToHost);

	cudaFree(d_array_a);
	cudaFree(d_array_b);
	cudaFree(d_array_c);

	for (int i = 0; i < ITEM_COUNT; ++i) {
		printf("result[%d]=%d\n", i, h_array_c[i]);
	}
	return 0;
}
{% endhighlight %}

从上面的程序可以看出来，CUDA的程序和一般的C语言程序并没有什么很大的区别。

一个CUDA程序的基本结构可以分为两个部分。

1. 在CPU上运行的串行程序。
2. 在GPU上运行的并行程序。



