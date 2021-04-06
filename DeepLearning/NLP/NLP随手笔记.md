# NLP随手笔记

[TOC]

### 

##### 1.TorchText提供数据集，在torchtext.datasets下，代码如下：

```python
import torchtext

train_data = torchtext.datasets.AG_NEWS(root="./data")
```

> 可能因为网络原因下载失败，可以在Colab上下载然后下载到本地；或访问ag_news.py中的两个URL下载。



2.对于一个Generator而言，要它生成符合条件的序列可以在每个时刻额外接入Context，更进一步，可以在每个时刻接入不同的Context，那便是Attention。此外，因为计算不是标准的RNN（标准的RNN只接收上一时刻的输出），将不能简单地对序列使用LSTM，而是需要使用LSTMCell并自行对时刻循环；而且，因为需要在适当时候停止生成，ED模型是不能批量处理数据的

##### 3.torch.LSTM的输出为output,hx,cx，其中output是每时刻的输出（最后时刻输出通过`output[:,-1,:]`获得），hx是各层每时刻的输出，cx是每层每时刻的细胞核值（一般没用）。如果只有一层，那么hx就是输出（`output[:,-1,:]`）

**input** of shape (seq_len, batch, input_size)

**h_0** of shape (num_layers * num_directions, batch, hidden_size)

**c_0** of shape (num_layers * num_directions, batch, hidden_size)