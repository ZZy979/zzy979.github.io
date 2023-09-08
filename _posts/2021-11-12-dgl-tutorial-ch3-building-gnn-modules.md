---
title: 【DGL教程】第3章 构建GNN模块
date: 2021-11-12 21:14:00 +0800
categories: [Graph Neural Network, DGL]
tags: [dgl, pytorch, graph neural network]
math: true
---
官方文档：<https://docs.dgl.ai/en/latest/guide/nn.html>

DGL的NN模块用于构建GNN模型，相当于PyTorch的NN模块或TensorFlow的层

以PyTorch后端为例，DGL NN模块的使用方法和PyTorch相同——**在构造函数中注册参数，在forward方法中进行张量运算**，因此可以和其他PyTorch的NN模块无缝集成，主要的不同是消息传递

`dgl.nn.pytorch.conv`包实现了一些常用的图卷积模块

## 1.DGL NN模块的构造函数
构造函数的工作通常包括：
* 设置选项（例如输出、输出和隐藏层的维数）
* 注册可学习的参数或子模块
* 重置（初始化）参数

## 2.DGL NN模块的forward方法
在NN模块中，`forward`方法执行实际的消息传递和计算

与PyTorch相比，DGL NN模块的`forward`方法接受一个额外的参数`dgl.DGLGraph`

`forward`方法的工作通常包括三部分：
* 对图进行检查
* 消息传递和归约（使用`update_all()`）
* 更新特征

官方文档以`SAGEConv`模块（GraphSAGE模型的实现）为例进行解析：
<https://docs.dgl.ai/en/latest/guide/nn-forward.html>

## 3.异构图卷积模块
`dgl.nn.pytorch.HeteroGraphConv`

官方文档：<https://docs.dgl.ai/en/latest/guide/nn-heterograph.html>

该模块是在异构图上运行DGL的NN模块的模块级别的封装，实现逻辑与消息传递API `multi_update_all()`相同（相当于`nn.Linear`和`F.linear()`的区别），包括：
* 在每个关系上运行NN模块
* 合并同一种顶点类型收到的来自多个关系的结果

用公式表示为

$$h_{dst}={AGG}_{r \in R(dst)}f_r(g_r,h_{src},h_{dst})$$

### 构造函数参数
```python
def __init__(self, mods, aggregate='sum'):
    super(HeteroGraphConv, self).__init__()
    self.mods = nn.ModuleDict(mods)
    # ...
    if isinstance(aggregate, str):
        self.agg_fn = get_aggregate_fn(aggregate)
    else:
        self.agg_fn = aggregate
    self.agg_fn = aggregate
```

边类型到NN模块的映射mods、用于聚集不同关系生成的顶点特征的函数agg，其中每个NN模块`forward()`函数的前两个参数应当是g和feat，g是关系二分图，feat是顶点输入特征或源顶点和目的顶点输入特征的二元组

### forward()方法
简化的`forward()`方法如下：

```python
def forward(self, g, inputs):
    outputs = {nty : [] for nty in g.dsttypes}
    for stype, etype, dtype in g.canonical_etypes:
        rel_graph = g[stype, etype, dtype]
        if rel_graph.number_of_edges() == 0:
            continue
        dstdata = self.mods[etype](
            rel_graph,
            (inputs[stype], inputs[dtype]))
        outputs[dtype].append(dstdata)
    rsts = {}
    for nty, alist in outputs.items():
        if len(alist) != 0:
            rsts[nty] = self.agg_fn(alist, nty)
    return rsts
```

输入异构图g和顶点类型到输入特征的映射inputs，对于每种边类型(stype, etype, dtype)，使用`mods[etype](g[stype, etype, dtype], inputs[stype])`计算dtype类型顶点的输出特征并添加到一个列表（边数为0的关系子图将被忽略），最后将每个顶点类型收到的所有结果使用agg聚集起来得到最终输出特征，返回顶点类型到输出特征的映射

### 聚集函数
接收两个参数：输出特征列表`List[tensor(N, d)]`和该顶点类型名称

内置聚集函数：

| 名称 | 实现逻辑 | 返回值 |
| --- | --- | --- |
| sum | torch.sum(torch.stack(outputs, dim=0), dim=0) | tensor(N, d) |
| mean | torch.mean(torch.stack(outputs, dim=0), dim=0) | tensor(N, d) |
| max | torch.max(torch.stack(outputs, dim=0), dim=0)[0] | tensor(N, d) |
| min | torch.min(torch.stack(outputs, dim=0), dim=0)[0] | tensor(N, d) |
| stack | torch.stack(outputs, dim=1) | tensor(N, R, d) |

自定义聚集函数示例：

```python
def agg(outputs, dtype):
    outputs = torch.stack(outputs, dim=1)  # (N, R, d)
    outputs = ... # aggregation (N, R, d) -> (N, d)
    return outputs
```

例如：异构图由(A, ab, B), (A, ac, C), (B, bc, C)三种关系构成，则输出为

```python
output = {
    'B': agg([mods['ab'](g['A', 'ab', 'B'], inputs['A'])]),
    'C': agg([mods['ac'](g['A', 'ac', 'C'], inputs['A']), mods['bc'](g['B', 'bc', 'C'], inputs['B']))
}
```
