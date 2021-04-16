# PyTorch笔记

[TOC]

##### 1.转换数据类型

使用`tensor.float()`、`tensor.long()`系列方法和`.type(other_var)`，但不建议使用`.type(dtype)`，无法导出ScriptModule。

##### 2.创建张量时维度可以为0，因此可以创建有形状可以cat但本身没有任何元素的张量

```python
import torch

sequence_length = 28
result = torch.empty(0, 3, 1, 1)

for i in range(sequence_length):
    output = torch.randn(1, 3, 1, 1)
    result = torch.cat([result, output], dim=0)
    
print(result.shape)
```

输出：

```python
torch.Size([28, 3, 1, 1])
```

##### 3.Python中的“前置声明”

```python
def f(i: 'A'):
    return i


class A:
    pass
```