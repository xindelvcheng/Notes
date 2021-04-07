# Onnx

[TOC]

## 一.入门

ONNX是通用模型表示格式（类似Web中的json），它还提供一个Runtime用于直接运行ONNX格式的模型。

### 1.安装

```bash
$ pip install onnx onnxruntime
```

### 2.使用`torch.onnx.export`函数导出模型

##### ①定义模型：

```python
class Model(torch.nn.Module):
    def __init__(self):
        super(Model, self).__init__()
        self.fc = torch.nn.Linear(784, 1000)

    def forward(self, x):
        return self.fc(x)
```

##### ②静态导出：

使用函数`torch.onnx.export(模型对象，输入参数，文件名或类文件对象，输出参数名称)`

```python
model = Model()
data = torch.randn(100, 784)

torch.onnx.export(model, data, "test.onnx")
```

可能的原理是运行一遍模型记录下计算图（因此需要转入数据）。网络的结构固定（不能包含if和for），模型的输入输出形状固定。

需要的变量（包括输入参数，各种超参数）可以指定名称，后面在运行时按名称传入（如果不指定则按forward方法参数的顺序传入）。

<font color=red>当参数不止一个的时候还是使用@torch.jit.script的时候会导致input_names中的name和模型中实际参数名不同（可以从input_data变成input_data.1，这时候可以通过export添加-verbose=True来判断</font>。

```python
torch.onnx.export(model, data, "test.onnx", input_names=["images"])
```

> 多个参数则在data的位置放一个元组，如果最后一个参数是一个元组，须在最后放一个空的字典（否则会被当成关键字参数列表）

需要注意的是，只会导出Model中forward函数中参与output的计算（作为PyTorchAPI中函数的参数）的那些参数，否则即便在forward函数的参数列表中也不会被导出（且这样会导致export时的参数和run时需要的参数不同）。

如果要让一个维度可变（例如batch_size不确定），则需要在dynamic_axis中指定：

```python
``dynamic_axes = {'input':[0]}``
```

这里即便维度只有1个也要给出列表，否则报错。

##### ③运行模型：

实例化onnxruntime.InferenceSession，调用其run方法。

```python
ort_inference_session = ort.InferenceSession("./test.onnx")
outputs = ort_inference_session.run(None, {"images": np.random.randn(100, 784).astype(np.float32)})
```

> 注意，这里数据是传给Model.forword方法而不是Model类的。

错误：RuntimeError: Input must be a list of dictionaries or a single numpy array for input '0'.：输入数据应是numpy.ndarray不能是torch.Tensor

##### ④.原则

构建网路时避免使用numpy，使用.detach代替.data

### 3.TorchScript

TorchScrip是PyTorch对模型的中间表示，可用于让模型运行在C++中。如果需要使用导出动态的onnx模型（即中间含有if或for让模型结构会发生变化的），则需要使用TorchScript。

使用torch.jit.trace，传入模型和参数，将执行该模型并记录为一个torch.jit.ScriptModule对象中的graph属性。

```python
graph(%self.1 : __torch__.MyCell,
      %input : Float(3, 4, strides=[4, 1], requires_grad=0, device=cpu),
      %h : Float(3, 4, strides=[4, 1], requires_grad=0, device=cpu)):
  %21 : __torch__.torch.nn.modules.linear.Linear = prim::GetAttr[name="fc"](%self.1)
  %23 : Tensor = prim::CallMethod[name="forward"](%21, %input)
  %14 : int = prim::Constant[value=1]() # C:/Users/wzzhang4/PycharmProjects/Temp/test1.py:16:0
  %15 : Float(3, 4, strides=[4, 1], requires_grad=1, device=cpu) = aten::add(%23, %h, %14) # C:/Users/wzzhang4/PycharmProjects/Temp/test1.py:16:0
  %16 : Float(3, 4, strides=[4, 1], requires_grad=1, device=cpu) = aten::tanh(%15) # C:/Users/wzzhang4/PycharmProjects/Temp/test1.py:16:0
  %17 : (Float(3, 4, strides=[4, 1], requires_grad=1, device=cpu), Float(3, 4, strides=[4, 1], requires_grad=1, device=cpu)) = prim::TupleConstruct(%16, %16)
  return (%17)
```

这是一个非常低级的表示，包含足够的信息但不便于阅读。可以通过.code可以查看对应的Python代码：

```python
def forward(self, input: Tensor, h: Tensor) -> Tuple[Tensor, Tensor]:
    _0 = torch.add((self.fc).forward(input, ), h, alpha=1)
    _1 = torch.tanh(_0)
    return (_1, _1)
```

这种模型是静态的。如果要获得静态的，则需要使用torch.jit.script方法。

## 二.Onnx规范

### 1.Op

(1)Gather

类似于pandas的iloc，用于索引和提取数据

