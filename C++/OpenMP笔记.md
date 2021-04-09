# OpenMP教程

[TOC]

OpenMP是用于并行计算的API，和需要GPU的Cuda，它可以利用CPU。在C++中使用时，它使用一些预处理指令给编译器信息，让程序具有并行的能力。这类似UE4中的那些空宏。

##### 1.开启simd

使用预编译指令

```
#pragma omp simd
```

速度对比：

```
Serial cost time:1.313000 s
OpenMP for cost time:0.102000 s
OpenMP simd cost time:0.351000 s
```

