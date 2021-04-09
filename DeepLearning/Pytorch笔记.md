# PyTorch笔记

[TOC]

##### 1.转换数据类型

使用`tensor.float()`、`tensor.long()`系列方法和`.type(other_var)`，但不建议使用`.type(dtype)`，无法导出ScriptModule。

