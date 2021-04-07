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

### 2.下载

torchtext.utils提供了两个工具方法

① download_from_url：用于下载文件（默认下载到当前文件夹的.data文件夹下），返回值为下载的文件的全路径

② extract_archive：用于解压缩文件（默认解压到压缩文件所在文件夹下），返回值为所有解压出的文件的全路径

