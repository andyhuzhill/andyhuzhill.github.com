---
layout: post
title: "学习CUDA GPU并行编程"
description: "CUDA 学习资料"
category: "Parallel Programming"
tags: [cuda, parallel programming]
---
{% include JB/setup %}
* TOC
{:toc}
<hr/>


[优达学城](http://www.youdaxue.com) 上开设了一门[并行编程入门](http://cn.udacity.com/course/intro-to-parallel-programming--cs344)的课程，使用的是Nvidia的CUDA并行编程环境。


在一般电脑用户的眼中，GPU(也就是常说的显卡)只是用来加速视频图像的显示或游戏视频的渲染，其实很早之前科学家们就想到了利用GPU的多核并行计算的能力来加快数据处理，Top500超级计算机榜单上的很多计算机就是采用了CPU+GPU的异构计算模型。 

我们每个普通人的电脑中装了支持CUDA或OpenCL之类的GPGPU(通用GPU计算)框架的显卡的话，就相当与一台小型的超级计算机了，这是多么令人激动的事情。 

不过如果想要利用起来这个微型“超级计算机”，已现在的技术还是得学习一下它底层使用的编程框架。我这里选择的是CUDA，因为CUDA有比较方便的编程环境和比较成熟的编程生态。Nvidia对CUDA的支持也比较好。


## CUDA 的学习资料

CUDA安装好之后，安装目录下有许多的帮助文档，初学者从这些文档开始学习是很方便的。

    ├── CUBLAS_Library.pdf.gz
    ├── CUDA_Binary_Utilities.pdf.gz
    ├── CUDA_C_Best_Practices_Guide.pdf.gz
    ├── CUDA_Compiler_Driver_NVCC.pdf.gz
    ├── CUDA_C_Programming_Guide.pdf.gz
    ├── CUDA_Debugger_API.pdf.gz
    ├── CUDA_Driver_API.pdf.gz
    ├── cuda-gdb.pdf.gz
    ├── CUDA_Installation_Guide_Linux.pdf.gz
    ├── CUDA_Installation_Guide_Mac.pdf.gz
    ├── CUDA_Installation_Guide_Windows.pdf.gz
    ├── CUDA_Math_API.pdf.gz
    ├── CUDA_Memcheck.pdf.gz
    ├── CUDA_Profiler_Users_Guide.pdf.gz
    ├── CUDA_Quick_Start_Guide.pdf.gz
    ├── CUDA_Runtime_API.pdf.gz
    ├── CUDA_Samples.pdf.gz
    ├── CUDA_Toolkit_Release_Notes.pdf.gz
    ├── CUDA_Video_Decoder.pdf.gz
    ├── CUFFT_Library.pdf.gz
    ├── CUPTI_Library.pdf.gz
    ├── CURAND_Library.pdf.gz
    ├── CUSOLVER_Library.pdf.gz
    ├── CUSPARSE_Library.pdf.gz
    ├── EULA.pdf.gz
    ├── Floating_Point_on_NVIDIA_GPU.pdf.gz
    ├── GPUDirect_RDMA.pdf.gz
    ├── Incomplete_LU_Cholesky.pdf.gz
    ├── Inline_PTX_Assembly.pdf.gz
    ├── Kepler_Tuning_Guide.pdf.gz
    ├── libdevice-users-guide.pdf.gz
    ├── libNVVM_API.pdf.gz
    ├── Maxwell_Compatibility_Guide.pdf.gz
    ├── Maxwell_Tuning_Guide.pdf.gz
    ├── NPP_Library.pdf.gz
    ├── Nsight_Eclipse_Edition_Getting_Started.pdf.gz
    ├── NVBLAS_Library.pdf.gz
    ├── NVRTC_User_Guide.pdf.gz
    ├── NVVM_IR_Specification.pdf.gz
    ├── Optimus_Developer_Guide.pdf.gz
    ├── ptx_isa_4.3.pdf.gz
    ├── PTX_Writers_Guide_To_Interoperability.pdf.gz
    └── Thrust_Quick_Start_Guide.pdf.gz

初学者 需要看的是  CUDA_C_Programming_Guide.pdf 和 CUDA_C_Best_Practices_Guide.pdf 两本。


网上也可以下载的到好几本关于CUDA的书籍

1. Programming Massively Parallel Processors
2. CUDA By Example (中文名叫《GPU高性能编程 CUDA 实战》)
3. Professional CUDA C Programming










