---
title: 【DGL】提取子图操作
date: 2021-11-14 18:55:23 +0800
categories: [Graph Neural Network, DGL]
tags: [dgl, pytorch, graph neural network]
---
`dgl.subgraph`和`dgl.sampling`模块定义了一些用于提取子图操作

官方文档：
* <https://docs.dgl.ai/en/latest/api/python/dgl.html#subgraph-extraction-ops>
* <https://docs.dgl.ai/en/latest/api/python/dgl.sampling.html>

示例图：

```python
>>> g = dgl.graph(([0, 0, 1, 2, 3], [1, 2, 3, 4, 4]))
>>> hg = dgl.heterograph({
    ('author', 'ap', 'paper'): ([0, 0, 1, 1, 2], [0, 1, 1, 2, 2]),
    ('paper', 'pc', 'conf'): ([0, 1, 2], [0, 0, 1])
})
```

![示例同构图](/assets/images/dgl-subgraph-extraction/示例同构图.png)

![示例异构图](/assets/images/dgl-subgraph-extraction/示例异构图.png)

## 1.顶点子图
```python
node_subgraph(graph, nodes, *, relabel_nodes=True, store_ids=True)
```

提取仅包含指定的顶点和这些顶点之间的边的子图

对于同构图，`nodes`是顶点id，可以是整型张量、整数可迭代对象或布尔张量(mask)；对于异构图，`nodes`是顶点类型到顶点id的映射

**提取出的顶点将从0开始重新编号**，顶点和边的原始id将分别保存在名为`dgl.NID`和`dgl.EID`的特征中；提取出的顶点和边的特征将被复制到子图中

```python
>>> sg = dgl.node_subgraph(g, [1, 3, 4])
>>> sg
Graph(num_nodes=3, num_edges=2,
      ndata_schemes={'_ID': Scheme(shape=(), dtype=torch.int64)}
      edata_schemes={'_ID': Scheme(shape=(), dtype=torch.int64)})
>>> sg.edges()
(tensor([0, 1]), tensor([1, 2]))
>>> sg.ndata[dgl.NID]
tensor([1, 3, 4])
```

![同构图顶点子图](/assets/images/dgl-subgraph-extraction/同构图顶点子图.png)

```python
>>> hsg = dgl.node_subgraph(hg, {'author': [1], 'paper': [1, 2], 'conf': [1]})
>>> hsg
Graph(num_nodes={'author': 1, 'conf': 1, 'paper': 2},
      num_edges={('author', 'ap', 'paper'): 2, ('paper', 'pc', 'conf'): 1},
      metagraph=[('author', 'paper', 'ap'), ('paper', 'conf', 'pc')])
>>> hsg.edges(etype='ap')
(tensor([0, 0]), tensor([0, 1]))
>>> hsg.edges(etype='pc')
(tensor([1]), tensor([0]))
>>> hsg.nodes['author'].data[dgl.NID]
tensor([1])
>>> hsg.nodes['paper'].data[dgl.NID]
tensor([1, 2])
>>> hsg.nodes['conf'].data[dgl.NID]
tensor([1])
```

![异构图顶点子图](/assets/images/dgl-subgraph-extraction/异构图顶点子图.png)

## 2.边子图
```python
edge_subgraph(graph, edges, *, relabel_nodes=True, store_ids=True)
```

提取仅包含指定的边的子图

对于同构图，`edges`是顶点id，可以是整型张量、整数可迭代对象或布尔张量(mask)；对于异构图，`edges`是边类型到边id的映射

**提取出的顶点将从0开始重新编号，删除孤立的顶点**，顶点和边的原始id将分别保存在名为`dgl.NID`和`dgl.EID`的特征中；提取出的顶点和边的特征将被复制到子图中

```python
>>> sg = dgl.edge_subgraph(g, [1, 3])
>>> sg
Graph(num_nodes=3, num_edges=2,
      ndata_schemes={'_ID': Scheme(shape=(), dtype=torch.int64)}
      edata_schemes={'_ID': Scheme(shape=(), dtype=torch.int64)})
>>> sg.edges()
(tensor([0, 1]), tensor([1, 2]))
>>> sg.ndata[dgl.NID]
tensor([0, 2, 4])
>>> sg.edata[dgl.EID]
tensor([1, 3])
```

![同构图边子图](/assets/images/dgl-subgraph-extraction/同构图边子图.png)

```python
>>> sg = dgl.edge_subgraph(g, [1, 3], relabel_nodes=False)
>>> sg
Graph(num_nodes=5, num_edges=2,
      ndata_schemes={}
      edata_schemes={'_ID': Scheme(shape=(), dtype=torch.int64)})
>>> sg.edges()
(tensor([0, 2]), tensor([2, 4]))
>>> sg.edata[dgl.EID]
tensor([1, 3])
```

![同构图边子图2](/assets/images/dgl-subgraph-extraction/同构图边子图2.png)

```python
>>> hsg = dgl.edge_subgraph(hg, {'ap': [0, 3], 'pc': [2]})
>>> hsg
Graph(num_nodes={'author': 2, 'conf': 1, 'paper': 2},
      num_edges={('author', 'ap', 'paper'): 2, ('paper', 'pc', 'conf'): 1},
      metagraph=[('author', 'paper', 'ap'), ('paper', 'conf', 'pc')])
>>> hsg.edges(etype='ap')
(tensor([0, 1]), tensor([0, 1]))
>>> hsg.edges(etype='pc')
(tensor([1]), tensor([0]))
>>> hsg.nodes['author'].data[dgl.NID]
tensor([0, 1])
>>> hsg.nodes['paper'].data[dgl.NID]
tensor([0, 2])
>>> hsg.nodes['conf'].data[dgl.NID]
tensor([1])
```

![异构图边子图](/assets/images/dgl-subgraph-extraction/异构图边子图.png)

## 3.入边子图
```python
in_subgraph(graph, nodes, *, relabel_nodes=False, store_ids=True)
```

提取指定的顶点及其入边构成的子图

对于同构图，`nodes`是顶点id，可以是整型张量或整数可迭代对象；对于异构图，`nodes`是顶点类型到顶点id的映射

**顶点不变**，边的原始id将保存在名为`dgl.EID`的特征中；提取出的顶点和边的特征将被复制到子图中
* `MultiLayerFullNeighborSampler`就是使用该函数实现的
* 对于同构图，`dgl.in_subgraph(g, nodes)`等价于`dgl.graph(g.in_edges(nodes), num_nodes=g.num_nodes())`

```python
>>> sg = dgl.in_subgraph(g, [3, 4])
>>> sg
Graph(num_nodes=5, num_edges=3,
      ndata_schemes={}
      edata_schemes={'_ID': Scheme(shape=(), dtype=torch.int64)})
>>> sg.edges()
(tensor([1, 2, 3]), tensor([3, 4, 4]))
>>> sg.edata[dgl.EID]
tensor([2, 3, 4])
```

![同构图入边子图](/assets/images/dgl-subgraph-extraction/同构图入边子图.png)

```python
>>> hsg = dgl.in_subgraph(hg, {'paper': [1, 2], 'conf': [0]})
>>> hsg
Graph(num_nodes={'author': 3, 'conf': 2, 'paper': 3},
      num_edges={('author', 'ap', 'paper'): 4, ('paper', 'pc', 'conf'): 2},
      metagraph=[('author', 'paper', 'ap'), ('paper', 'conf', 'pc')])
>>> hsg.edges(etype='ap')
(tensor([0, 1, 1, 2]), tensor([1, 1, 2, 2]))
>>> hsg.edges(etype='pc')
(tensor([0, 1]), tensor([0, 0]))
>>> hsg.edges['ap'].data[dgl.EID]
tensor([1, 2, 3, 4])
>>> hsg.edges['pc'].data[dgl.EID]
tensor([0, 1])
```

![异构图入边子图](/assets/images/dgl-subgraph-extraction/异构图入边子图.png)

## 4.出边子图
```python
out_subgraph(graph, nodes, *, relabel_nodes=False, store_ids=True)
```

提取指定的顶点及其出边构成的子图

对于同构图，`nodes`是顶点id，可以是整型张量或整数可迭代对象；对于异构图，`nodes`是顶点类型到顶点id的映射

**顶点不变**，边的原始id将保存在名为dgl.EID的特征中；提取出的顶点和边的特征将被复制到子图中

```python
>>> sg = dgl.out_subgraph(g, [0, 3])
>>> sg
Graph(num_nodes=5, num_edges=3,
      ndata_schemes={}
      edata_schemes={'_ID': Scheme(shape=(), dtype=torch.int64)})
>>> sg.edges()
(tensor([0, 0, 3]), tensor([1, 2, 4]))
>>> sg.edata[dgl.EID]
tensor([0, 1, 4])
```

![同构图出边子图](/assets/images/dgl-subgraph-extraction/同构图出边子图.png)

```python
>>> hsg = dgl.out_subgraph(hg, {'author': [0], 'paper': [1, 2]})
>>> hsg
Graph(num_nodes={'author': 3, 'conf': 2, 'paper': 3},
      num_edges={('author', 'ap', 'paper'): 2, ('paper', 'pc', 'conf'): 2},
      metagraph=[('author', 'paper', 'ap'), ('paper', 'conf', 'pc')])
>>> hsg.edges(etype='ap')
(tensor([0, 0]), tensor([0, 1]))
>>> hsg.edges(etype='pc')
(tensor([1, 2]), tensor([0, 1]))
>>> hsg.edges['ap'].data[dgl.EID]
tensor([0, 1])
>>> hsg.edges['pc'].data[dgl.EID]
tensor([1, 2])
```

![异构图出边子图](/assets/images/dgl-subgraph-extraction/异构图出边子图.png)

## 5.顶点类型子图
```python
node_type_subgraph(graph, ntypes)
```

提取仅包含指定类型的顶点和这些顶点之间的边的子图
`ntypes`是顶点类型列表

提取出的顶点和边的特征将被复制到子图中

```python
>>> hsg = dgl.node_type_subgraph(hg, ['author', 'paper'])
>>> hsg
Graph(num_nodes={'author': 3, 'paper': 3},
      num_edges={('author', 'ap', 'paper'): 5},
      metagraph=[('author', 'paper', 'ap')])
>>> hsg.edges(etype='ap')
(tensor([0, 0, 1, 1, 2]), tensor([0, 1, 1, 2, 2]))
```

![顶点类型子图](/assets/images/dgl-subgraph-extraction/顶点类型子图.png)

## 6.边类型子图
```python
edge_type_subgraph(graph, etypes)
```

提取仅包含指定类型的边的子图

`etypes`是边类型列表

提取出的顶点和边的特征将被复制到子图中

```python
>>> hsg = dgl.edge_type_subgraph(hg, ['pc'])
>>> hsg
Graph(num_nodes={'conf': 2, 'paper': 3},
      num_edges={('paper', 'pc', 'conf'): 3},
      metagraph=[('paper', 'conf', 'pc')])
>>> hsg.edges(etype='pc')
(tensor([0, 1, 2]), tensor([0, 0, 1]))
```

![边类型子图](/assets/images/dgl-subgraph-extraction/边类型子图.png)

## 7.邻居采样子图
```python
dgl.sampling.sample_neighbors(g, nodes, fanout, prob=None, copy_ndata=True, copy_edata=True)
```

采样指定顶点的邻边，返回原图中**所有顶点**和采样的边构成的子图

对于同构图，`nodes`是顶点id；对于异构图，`nodes`是顶点类型到顶点id的映射

`fanout`是扇出系数，是指每个顶点在每种边类型上采样边的数量，可以是一个整数或边类型到整数的映射，-1表示不采样（选择所有边）

`prob`是用作采样概率的边特征名称

`copy_ndata`指定是否复制原图的顶点特征

`copy_edata`指定是否复制原图的边特征

边的原始id将保存在名为`dgl.EID`的特征中

`MultiLayerNeighborSampler`就是使用该函数实现的

```python
>>> sg = dgl.sampling.sample_neighbors(g, [4], 1)
>>> sg
Graph(num_nodes=5, num_edges=1,
      ndata_schemes={}
      edata_schemes={'_ID': Scheme(shape=(), dtype=torch.int64)})
>>> sg.edges()
(tensor([2]), tensor([4]))
```

![同构图邻居采样子图](/assets/images/dgl-subgraph-extraction/同构图邻居采样子图.png)

```python
>>> hsg = dgl.sampling.sample_neighbors(hg, {'paper': [1, 2], 'conf': [0]}, 1)
>>> hsg
Graph(num_nodes={'author': 3, 'conf': 2, 'paper': 3},
      num_edges={('author', 'ap', 'paper'): 2, ('paper', 'pc', 'conf'): 1},
      metagraph=[('author', 'paper', 'ap'), ('paper', 'conf', 'pc')])
>>> hsg.edges(etype='ap')
(tensor([0, 2]), tensor([1, 2]))
>>> sg.edges(etype='pc')
(tensor([1]), tensor([0]))
```

![异构图邻居采样子图](/assets/images/dgl-subgraph-extraction/异构图邻居采样子图.png)
