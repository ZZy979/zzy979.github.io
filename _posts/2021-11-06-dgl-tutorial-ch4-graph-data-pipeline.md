---
title: 【DGL教程】第4章 图数据集
date: 2021-11-06 16:28:52 +0800
categories: [Graph Neural Network, DGL]
tags: [dgl, pytorch, graph neural network]
---
官方文档：<https://docs.dgl.ai/en/latest/guide/data.html>

`dgl.data`实现了很多常用的图数据集，这些数据集都是`dgl.data.DGLDataset`的子类

DGL官方推荐通过继承`dgl.data.DGLDataset`来实现自己的数据集，从而可以更方便地加载、处理、保存图数据集

## 1.DGLDataset类
dgl.data.DGLDataset类处理数据集的流程包括以下几步：下载、处理、保存到磁盘、从磁盘加载，如下图所示

![图数据集流程图](https://data.dgl.ai/asset/image/userguide_data_flow.png)

自定义数据集类：

```python
from dgl.data import DGLDataset

class MyDataset(DGLDataset):
    def __init__(self):
        super().__init__(name='my_dataset', url='https://example.com/path/to/my_dataset.zip')

    def download(self):
        # download raw data to local disk
        pass

    def save(self):
        # save processed data to directory `self.save_path`
        pass

    def load(self):
        # load processed data from directory `self.save_path`
        pass

    def process(self):
        # process raw data to graphs, labels, splitting masks
        pass

    def has_cache(self):
        # check whether there are processed data in `self.save_path`
        pass

    def __getitem__(self, idx):
        # get one example by index
        pass

    def __len__(self):
        # number of data examples
        pass
```

其中`process()`, `__getitem__(idx)`和`__len__()`是必须实现的方法

`DGLDataset`类的目的是提供一种标准、方便的加载图数据的方式，可以存储图、特征、标签、划分以及数据集的其他基本信息（如类别数）

下面介绍实现数据集中方法的最佳实践

### 1.1 下载原始数据
`DGLDataset.download()`方法用于从`self.url`指定的URL下载原始数据，并保存到`self.raw_dir`目录
* DGL提供了一个辅助函数`dgl.data.utils.download()`用于从指定的URL下载文件
* `DGLDataset`的`raw_dir`属性是原始数据下载目录，如果在构造函数中指定了`raw_dir`参数则使用指定的目录，如果未指定则使用环境变量DGL_DOWNLOAD_DIR指定的目录，如果该环境变量不存在则默认为~/.dgl
* `DGLDataset`的`raw_path`属性是`os.path.join(self.raw_dir, self.name)`（也可以在子类中覆盖），可用于原始数据的解压目录（如果原始数据是zip文件）

示例：

```python
def download(self):
    zip_file_path = os.path.join(self.raw_dir, 'my_dataset.zip')
    download(self.url, path=zip_file_path)
    extract_archive(zip_file_path, self.raw_path)
```

### 1.2 处理数据
`DGLDataset.process()`方法用于将`self.raw_dir`或`self.raw_path`中的原始数据处理成`DGLGraph`的格式，一般包括读取原始数据、数据清洗、构造图、读取顶点特征和标签以及划分数据集等步骤，具体逻辑取决于原始数据的格式（可能是pkl, npz, mat, csv, txt等等），这是需要自己实现的主要部分（也是最麻烦的部分）

以只有一个图的数据集为例（例如顶点分类数据集），基本框架如下：

```python
def process(self):
    data = _read_raw_data(self.raw_path)
    data = _clean(data)
    g = dgl.graph(...)
    g.ndata['feat'] = ...
    g.ndata['label'] = ...
    g.ndata['train_mask'] = ...
    g.ndata['val_mask'] = ...
    g.ndata['test_mask'] = ...
    self.g = g

def __getitem__(self, idx):
    if idx != 0:
        raise IndexError('This dataset has only one graph')
    return self.g

def __len__(self):
    return 1
```

注：
* `_read_raw_data()`和`_clean()`是需要自己实现的读取原始数据和清洗数据的逻辑
* 该示例只有一个同构图，实际也可能是异构图，也可能包含多个图（例如图分类数据集）

示例：<https://github.com/ZZy979/pytorch-tutorial/tree/master/gnn/data>

### 1.3 保存和加载数据
DGL推荐实现数据集的`save()`和`load()`方法，将预处理完的数据缓存到磁盘，下次使用时可直接从磁盘加载，不需要再执行`process()`方法，`has_cache()`方法返回磁盘上是否有缓存的已处理好的数据

DGL提供了4个函数：
* `dgl.save_graphs()`和`dgl.load_graphs()`用于向磁盘保存/从磁盘读取`DGLGraph`对象
* `dgl.data.utils.save_info()`和`dgl.data.utils.load_info()`用于向磁盘保存/从磁盘读取数据集的相关信息（实际上就是`pickle.dump()`和`pickle.load()`）

保存路径：
* `DGLDataset`的`save_dir`属性是处理好的数据的保存目录，如果在构造函数中指定了`save_dir`参数则使用指定的目录，否则默认为`raw_dir`
* `DGLDataset`的`save_path`属性是`os.path.join(self.save_dir, self.name)`（也可以在子类中覆盖），一般将处理好的数据保存到`save_path`目录下

典型用法：

（1）顶点分类数据集（只有一个图）

```python
def save(self):
    # save graphs and labels
    graph_path = os.path.join(self.save_path, self.name + '_dgl_graph.bin')
    save_graphs(graph_path, [self.g])

def load(self):
    # load processed data from directory `self.save_path`
    graph_path = os.path.join(self.save_path, self.name + '_dgl_graph.bin')
    graphs, _ = load_graphs(graph_path)
    self.labels = label_dict['labels']
    self.g = graphs[0]

def has_cache(self):
    # check whether there are processed data in `self.save_path`
    graph_path = os.path.join(self.save_path, self.name + '_dgl_graph.bin')
    return os.path.exists(graph_path)
```

（2）图分类数据集（包含多个图）

```python
def save(self):
    # save graphs and labels
    graph_path = os.path.join(self.save_path, self.name + '_dgl_graph.bin')
    save_graphs(graph_path, self.graphs, {'labels': self.labels})
    # save other information in python dict
    info_path = os.path.join(self.save_path, self.name + '_info.pkl')
    save_info(info_path, {'num_classes': self.num_classes})

def load(self):
    # load processed data from directory `self.save_path`
    graph_path = os.path.join(self.save_path, self.name + '_dgl_graph.bin')
    self.graphs, label_dict = load_graphs(graph_path)
    self.labels = label_dict['labels']
    info_path = os.path.join(self.save_path, self.name + '_info.pkl')
    self.num_classes = load_info(info_path)['num_classes']

def has_cache(self):
    # check whether there are processed data in `self.save_path`
    graph_path = os.path.join(self.save_path, self.name + '_dgl_graph.bin')
    info_path = os.path.join(self.save_path, self.name + '_info.pkl')
    return os.path.exists(graph_path) and os.path.exists(info_path)
```

## 2.使用图数据集
### 2.1 图分类数据集
图分类数据集和传统机器学习的数据集类似，包含了一组样本和对应的标签，只是每个样本是一个`dgl.DGLGraph`，标签是一个张量，样本的特征保存在不同的顶点特征或边特征中

下面以QM7b数据集为例演示使用方法

创建数据集

```python
>>> from dgl.data import QM7bDataset
>>> qm7b = QM7bDataset()  # 首次使用会先下载数据集
>>> len(qm7b)
7211
>>> qm7b.num_labels
14
>>> g, label = qm7b[0]
>>> g
Graph(num_nodes=5, num_edges=25,
      ndata_schemes={}
      edata_schemes={'h': Scheme(shape=(1,), dtype=torch.float32)})
>>> g.edata
{'h': tensor([[36.8581],
        [ 2.8961],
        ...
        [ 0.5000]])}
>>> label
tensor([-4.2093e+02,  3.9695e+01,  6.2184e-01, -1.6013e+01,  4.1620e+00,
         3.6768e+01,  1.5725e+01, -3.9861e+00, -1.0949e+01,  1.3230e-01,
        -1.4134e+01,  1.0870e+00,  2.5346e+00,  2.4322e+00])
```

可以看到该数据集共有7211个样本，每个样本有14个标签（对应14个预测任务），第1个样本图有5个顶点、25条边，有一个名为h的边特征，维数为1

遍历数据集

可以使用PyTorch的`DataLoader`遍历数据集

```python
from torch.utils.data import DataLoader

## load data
dataset = QM7bDataset()
num_labels = dataset.num_labels

## create collate_fn
def _collate_fn(batch):
    graphs, labels = batch
    g = dgl.batch(graphs)
    labels = torch.tensor(labels, dtype=torch.long)
    return g, labels

## create dataloaders
dataloader = DataLoader(dataset, batch_size=1, shuffle=True, collate_fn=_collate_fn)

## training
for epoch in range(100):
    for g, labels in dataloader:
        # your training code here
        pass
```

### 2.2 顶点分类数据集
顶点分类通常只在一个图上进行，因此这类数据集只有一个图，样本特征和标签保存在顶点特征中

以Citeseer数据集为例，该数据集包含一个图，有3327个顶点、9228条边，特征、标签、训练集、验证集、测试集掩码分别在顶点特征feat, label, train_mask, val_mask, test_mask中，顶点特征为3703维，6个类别（标签范围为[0, 5]）

```python
>>> from dgl.data import CiteseerGraphDataset
>>> citeseer = CiteseerGraphDataset()
>>> len(citeseer)
1
>>> citeseer.num_classes
6
>>> g = citeseer[0]
>>> g
Graph(num_nodes=3327, num_edges=9228,
      ndata_schemes={'train_mask': Scheme(shape=(), dtype=torch.bool), 'val_mask': Scheme(shape=(), dtype=torch.bool), 'test_mask': Scheme(shape=(), dtype=torch.bool), 'label': Scheme(shape=(), dtype=torch.int64), 'feat': Scheme(shape=(3703,), dtype=torch.float32)}
      edata_schemes={})
>>> g.ndata['feat'].shape
torch.Size([3327, 3703])
>>> g.ndata['label'].shape
torch.Size([3327])
>>> g.ndata['label'][:10]
tensor([3, 1, 5, 5, 3, 1, 3, 0, 3, 5])
>>> train_idx = torch.nonzero(g.ndata['train_mask']).squeeze()
>>> train_set = g.ndata['feat'][train_idx]
>>> train_set.shape
torch.Size([120, 3703])
```

### 2.3 连接预测数据集
连接预测数据集和顶点分类数据集类似，也只有一个图，但训练集、验证集、测试集掩码在边特征中，这类数据集有`dgl.data.KnowledgeGraphDataset`的几个子类

### 2.4 OGB数据集
Open Graph Benchmark (OGB): <https://ogb.stanford.edu/docs/home/>
