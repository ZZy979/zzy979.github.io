---
title: 【DGL教程】第1章 图(Graph)
date: 2021-04-23 15:45:52 +0800
categories: [DGL]
tags: [dgl, pytorch, graph neural network]
---
Deep Graph Library (DGL)是一个用于构建图神经网络模型的框架
* 网址：<https://www.dgl.ai/>
* 官方文档：<https://docs.dgl.ai/>
* 论坛：<https://discuss.dgl.ai/>

## 安装
* CPU版本：pip install dgl -f https://data.dgl.ai/wheels/repo.html
* CUDA 11.0：pip install dgl-cu110 -f https://data.dgl.ai/wheels/repo.html

DGL支持PyTorch, MXNet和TensorFlow三种后端，默认为PyTorch，可在~/.dgl/config.json中配置

本章官方文档：<https://docs.dgl.ai/guide/graph.html>

## 1.基本概念
图由顶点集合和边集合构成，表示为G=(V, E)

DGL表示图的核心数据结构是`dgl.DGLGraph`

DGL用整数表示一个顶点，叫做**顶点ID**；用一对整数(u, v)表示一条边，分别对应边的起点和终点的顶点ID，同时每条边也有一个**边ID**；在DGL中边都是有向的

DGL使用长度为n的一维张量来表示n个顶点，叫做**顶点张量**；使用两个顶点张量的元组(U, V)来表示n条边，其中(U[i], V[i])表示第i条边

### 创建图
```python
>>> import dgl
>>> import torch
>>> u, v = torch.tensor([0, 0, 0, 1]), torch.tensor([1, 2, 3, 3])
>>> g = dgl.graph((u, v))
>>> g
Graph(num_nodes=4, num_edges=4,
      ndata_schemes={}
      edata_schemes={})
>>> g.num_nodes()
4
>>> g.num_edges()
4
>>> g.nodes()
tensor([0, 1, 2, 3])
>>> g.edges()
(tensor([0, 0, 0, 1]), tensor([1, 2, 3, 3]))
```

![创建图](https://data.dgl.ai/asset/image/user_guide_graphch_1.png)

其中`u`和`v`可以是PyTorch张量，也可以是Python的整数可迭代对象（例如列表）

（`g`的实际类型是`dgl.DGLHeteroGraph`，在最新版本中与`dgl.DGLGraph`等价）

上面的例子中顶点ID的范围是从边中推断出来的，如果具有最大ID的顶点是孤立的（没有关联的边），则需要指定顶点个数：

```python
>>> g = dgl.graph((u, v), num_nodes=8)
```

`dgl.to_bidirected()`将有向图转换为无向图

```python
>>> bg = dgl.to_bidirected(g)
>>> bg.edges()
(tensor([0, 0, 0, 1, 1, 2, 3, 3]), tensor([1, 2, 3, 0, 3, 0, 0, 1]))
```

### 图的可视化

```python
import networkx as nx
import matplotlib.pyplot as plt
# Since the actual graph is undirected, we convert it for visualization purpose.
nx_g = g.to_networkx().to_undirected()
# Kamada-Kawaii layout usually looks pretty for arbitrary graphs
pos = nx.kamada_kawai_layout(nx_g)
nx.draw(nx_g, pos, with_labels=True, node_color=[[.7, .7, .7]])
plt.show()
```

完整操作列表：<https://docs.dgl.ai/api/python/dgl.DGLGraph.html>

## 2.顶点和边的特征
顶点和边都可以具有若干自定义名字的**特征**，分别通过`ndata`和`edata`属性访问

下面的代码创建了两个顶点特征x（长度为3的1维向量）和y（标量），以及一个边特征x（标量）

```python
>>> import dgl
>>> import torch
>>> g = dgl.graph(([0, 0, 1, 5], [1, 2, 2, 0]))  # 6 nodes, 4 edges
>>> g
Graph(num_nodes=6, num_edges=4,
      ndata_schemes={}
      edata_schemes={})
>>> g.ndata['x'] = torch.ones(g.num_nodes(), 3)  # node feature of length 3
>>> g.edata['x'] = torch.ones(g.num_edges(), dtype=torch.int32)  # scalar integer feature
>>> g
Graph(num_nodes=6, num_edges=4,
      ndata_schemes={'x': Scheme(shape=(3,), dtype=torch.float32)}
      edata_schemes={'x': Scheme(shape=(), dtype=torch.int32)})
>>> # different names can have different shapes
>>> g.ndata['y'] = torch.randn(g.num_nodes(), 5)
>>> g.ndata['x'][1]   # get node 1's feature
tensor([1., 1., 1.])
>>> g.edata['x'][torch.tensor([0, 3])]  # get features of edge 0 and 3
tensor([1, 1], dtype=torch.int32)
```

关于顶点和边特征：
* 特征只能是数值（整数/浮点数），可以是标量、向量或多维张量，名字可以任意指定
* 给顶点特征赋值时，第一维大小必须等于顶点数；给边特征赋值时，第一维大小必须等于边数
* 要实现加权图，将边的权重作为边特征即可

## 3.异构图
异构图是具有不同类型的顶点和边的图
在DGL中，不同类型的顶点/边有独立的ID范围和特征存储，如下图所示

![异构图](https://data.dgl.ai/asset/image/user_guide_graphch_2.png)

在DGL中，异构图是由一系列二分子图指定的，每个二分子图的起点类型、边类型、终点类型分别相同，由一个**关系**指定，关系是一个字符串三元组：(起点类型, 边类型, 终点类型)，例如('user', 'plays', 'game')

构造异构图的方法是指定一系列关系及其顶点张量对：

```
{
    relation1 : (src_node_tensor1, dst_node_tensor1),
    relation2 : (src_node_tensor2, dst_node_tensor2),
    ...
}
```

例如，下面的代码构造了一个包含“药物”、“基因”、“疾病”三种顶点和“反应”、“作用”、“治疗”三种边的异构图

```python
>>> import dgl
>>> import torch
>>> # Create a heterograph with 3 node types and 3 edges types.
>>> g = dgl.heterograph({
    ('drug', 'interacts', 'drug'): (torch.tensor([0, 1]), torch.tensor([1, 2])),
    ('drug', 'interacts', 'gene'): (torch.tensor([0, 1]), torch.tensor([2, 3])),
    ('drug', 'treats', 'disease'): (torch.tensor([1]), torch.tensor([2]))
})
>>> g
Graph(num_nodes={'disease': 3, 'drug': 3, 'gene': 4},
      num_edges={('drug', 'interacts', 'drug'): 2, ('drug', 'interacts', 'gene'): 2, ('drug', 'treats', 'disease'): 1},
      metagraph=[('drug', 'drug', 'interacts'), ('drug', 'gene', 'interacts'), ('drug', 'disease', 'treats')])
```

访问异构图中的顶点和边需要指定类型
注意：如果边类型的名字唯一，则可仅使用边类型的名字，否则必须使用关系三元组

```python
>>> g.ntypes
['disease', 'drug', 'gene']
>>> g.etypes
['interacts', 'interacts', 'treats']
>>> g.canonical_etypes
[('drug', 'interacts', 'drug'), ('drug', 'interacts', 'gene'), ('drug', 'treats', 'disease')]
>>> g.num_nodes()
10
>>> g.num_nodes('drug')
3
>>> g.nodes()
DGLError('Node type name must be specified if there are more than one '
>>> g.nodes('drug')
tensor([0, 1, 2])
>>> g.num_edges()
5
>>> g.num_edges(('drug', 'interacts', 'drug'))
2
>>> g.edges['interacts']
dgl._ffi.base.DGLError: Edge type "interacts" is ambiguous. Please use canonical edge type in the form of (srctype, etype, dsttype)
>>> g.num_edges(('drug', 'treats', 'disease'))
1
>>> g.num_edges('treats')
1
```

同构图和二分图就是只有一个关系的特殊异构图，同构图的起点类型和终点类型也相同，而二分图的起点类型和终点类型不同

```python
>>> # A homogeneous graph
>>> dgl.heterograph({('node_type', 'edge_type', 'node_type'): (u, v)})
>>> # A bipartite graph
>>> dgl.heterograph({('source_type', 'edge_type', 'destination_type'): (u, v)})
```

访问异构图中顶点和边的特征要使用新的语法：`g.nodes['node_type'].data['name']`和`g.edges['edge_type'].data['name']`

```python
>>> # Set/get feature 'hv' for nodes of type 'drug'
>>> g.nodes['drug'].data['hv'] = torch.ones(g.num_nodes('drug'))
g.nodes['drug'].data['hv']
tensor([1., 1., 1.])
>>> # Set/get feature 'he' for edge of type 'treats'
>>> g.edges['treats'].data['he'] = torch.zeros(g.num_edges(('drug', 'treats', 'disease')))
>>> g.edges['treats'].data['he']
tensor([0.])
```

将异构图转换为同构图：`dgl.to_homogeneous()`
