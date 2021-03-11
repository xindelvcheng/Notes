# NLP笔记

[TOC]

## 一.数据集

### 1.TorchText

TorchText提供数据集，在torchtext.datasets下，代码如下：

```python
import torchtext

train_data = torchtext.datasets.AG_NEWS(root="./data")
```

> 可能因为网络原因下载失败，可以在Colab上下载然后下载到本地；或访问ag_news.py中的两个URL下载。



