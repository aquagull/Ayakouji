---
layout: math
title: "Ring AllReduce"
date: 2024-10-31
categories: [distributed training]
---

## 传统多卡并行

假设我们有$N$GPU进行并行训练，每张GPU上的模型梯度大小为$K$。在训练中，我们需要计算所有GPU上的梯度的平均值，然后更新参数。为此，最简单的方法，就是指定一张GPU为master，其余GPU为slave。过程是，反向传播后，master汇聚所有slave的梯度，求平均值，然后将结果发回slave。

![](https://blog-assets.unvs.cc/2021/05/ring-allreduce-fig1.webp)

这里，我们计算一下每张GPU上的发送和接受的数据量大小：
- 对于master：
    - 由于所有slave都需要发送完整梯度给master，因此收到的数据量为$(N - 1) \cdot K$。
    - master还需要将reduce后的梯度发送到每个slave上，因此其发送的数据量也为$(N - 1) \cdot K$。

- 对于slave：
    - 发送和接受的数据量均为$K$。

这种简单的并行方式有两个缺点：
1. master和slave的通信量不一样，然而在实际中，GPU之间的通信带宽是相差不大的。这种情况下，master会成为一个通信瓶颈。
2. 每个GPU的计算量是不均等的，master还负责对梯度求平均值，在此时间段，slave只能空转。

Pytorch中的`torch.nn.DataParallel`就是采用这种方式在GPU之间汇聚和发送梯度的。在GPU数量较多的情况下，通信会成为瓶颈，加速比会远低于$N$。

## Ring AllReduce 多卡并行
在2017年，百度引入了Ring AllReduce算法，解决了上面两个缺点。

在该算法中，所有GPU组成一个环。算法总共包含两个步骤，分别为scatter-reduce和allgather。

![](https://blog-assets.unvs.cc/2021/05/ring-allreduce-fig2.webp)