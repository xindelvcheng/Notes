# Encoder-Decoder模型导出笔记

[TOC]

##### 1.避免在模型中使用numpy和math操作，可能会失效也可能无法导出

例如使用np.random.rand()会变成常量：

```python
class Net(torch.nn.Module):
    def __init__(self):
        super(Net, self).__init__()

    def forward(self, x):
        noise = np.random.rand()
        return x + noise
```

导出时noise变成：

```
%1 : Float(requires_grad=0, device=cpu) = onnx::Constant[value={0.124173}]()
```

使用math.sqrt同样也不能导出，其结果会变成常量，这种情况会报警告：converting a tensor to a python float might cause the trace to be incorrect。

使用np.random.randn()则会无法导出，且报出意义不明的错误

```python
class Net(torch.nn.Module):
    def __init__(self):
        super(Net, self).__init__()

    def forward(self, x):
        noise = np.random.randn(1, 1)
        return x + noise
```

报错：

```
RuntimeError: output 1 ( 1.6503
[ CPUDoubleType{1,1} ]) of traced region did not have observable data dependence with trace inputs; this probably indicates your program cannot be understood by the tracer.
```

##### 2.有些地方不能使用元组代替列表

例如torch.onnx.export中的input_names

##### 3.有时候因为未知的原因，onnx导出的模型会凭空插入一些类型转换导致运算符两侧张量数据类型不一致而失败

##### 不能使用的高级API

截至opset_version=12明确不能使用的有：

torch.linspace

torch.broadcast_tensors

iter()

random.random

