# PyTorch随手笔记

[TOC]



##### 1.torch.nn.Embedding用于转换单个词/句子为编码或编码序列

```python
embedding_layer = torch.nn.Embedding(1000,15)xxxxxxxxxx embedding_layer = torch.nn.Ebeembedding_layer = torch.nn.Embedding(1000,15)
```

转换单个词：

```python
word_index = torch.tensor(1)
embedding_layer(word_index)
```

转换一个句子：

```python
embedding_layer(torch.LongTensor([0,1,2]))
```

转换一个批次的句子：

```python
embedding_layer(torch.LongTensor([[0,1,2],[3,2,1]]))
```

##### 2.matual,mm和bmm函数

mm：用于矩阵乘法

bmm：用于多批次数据在同批次间进行矩阵乘法

matmul：会根据输入张量的维度视情况进行内积、矩阵乘法、批次矩阵乘法