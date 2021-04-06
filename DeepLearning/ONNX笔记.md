# ONNX笔记

[TOC]

一.

## 二.常见节点

##### 1.Cast

用于转换数据类型，示例如下：

```onnx
onnx::Cast[to=1](%input)
```

其中to为枚举TensorProtoDataType，从1开始，依次为FLOAT、UINT8、INT8、UINT16、INT16、INT32、INT64、STRING、BOOL、FLOAT16、DOUBLE、UINT32、UINT64、COMPLEX64。例如to=1意为将input的dtype转成FLOAT（32位浮点），to=7意为将input的dtype转为INT64。

