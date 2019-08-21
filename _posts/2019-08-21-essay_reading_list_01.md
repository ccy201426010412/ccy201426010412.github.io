---
layout:     post
title:      文集补校(一)
subtitle:   卷积神经网络结构优化综述
date:       2019-08-13
author:     王鹏程
header-img: img/post-bg-ios10.jpg
catalog: true
tags:
    - 卷积神经网络结构优化综述
    - Linux
    - 优化
    - 神经网络
    - 论文笔记
---

# 卷积神经网络结构优化综述

**参考文献格式**

[[1]林景栋,吴欣怡,柴毅,尹宏鹏.卷积神经网络结构优化综述[J/OL].自动化学报:1-14[2019-08-21].https://doi.org/10.16383/j.aas.c180275.](http://kns.cnki.net/KCMS/detail/11.2109.TP.20190710.1703.009.html)

## 1. 网路剪枝与稀疏化

_参考文献：_ [Network trimming: a
data-driven neuron pruning approach towards efficient deep
architectures](https://arxiv.org/abs/1607.03250)
卷积神经网络从卷积层到全连接层存在大量的冗余参数,当参数趋近于0的时候可以进行参数裁剪，剔除掉不重
要的连接、节点甚至卷积核，以达到精简网络结构的目的.好处如下：

- 有效缓解了过拟合现象的发生
- 稀疏网络在以 CSR (Compressed sparse row format, CSR) 和 CSC (Compressed sparse column format)等稀疏矩阵存储格式存储于计算机中可大幅降低内存开销。
- 训练参数的减少使得网络训练阶段和预测阶段花费时间更少。

主要方法：

- 训练中稀疏约束
    + Collins等在参数空间中通过贪婪搜索决定需要稀疏化的隐含层([Memory bounded deep convolutional networks](https://arxiv.org/abs/1412.1442))
    + 迭代硬阈值 (Iter-ative hard thresholding, IHT)([Training skinny deep
neural networks with iterative hard thresholding methods](https://arxiv.org/abs/1607.05423)):
       * 第一步中剔除隐含节点间权值较小的连接 , 然后微调 (Fine-tune) 其他重要的卷积核.
       * 第二步中激活断掉的连接,重新训练整个网络以获取更有用的特征
    + 前向–后向切分法(Forward-backward splitting method)([19]):
    + 结构化稀疏学习(Structured sparsity learning, SSL)([20]):接学习到的硬件友好型稀疏网络不仅具有更加紧凑的结构 , 而且运行速度可提升 3 倍至 5 倍.
    + 以分组形式剪枝卷积核输入,以数据驱动的方式获取最优感受野 (Receptive field)([21])
    + 利用一系列优化措施将不可微分的 l 0 范数正则项加入到目标函数 ,学习到的稀疏网络不仅具有良好的泛化性能 , 而且极大加速了模型训练和推导过程。([22])
    + Dropout 作为一种强有力的网络优化方法 , 可被视为特殊的正则化方法 , 被广泛用于防止网络训练过拟合[23-24]
    + 自适应 Dropout:根据特征和神经元的分布使用不同的多项式采样方式 , 其收敛速度相对于标准Dropout 提高 50 %.[25]
- 训练后剪枝:从已有模型入手，消除网络中的冗余信息。由剪枝粒度不同划分如下[26]：
    + 层间剪枝:减少网络深度
    + 特征图剪枝:减少网络宽度
    + kxk核剪枝:减少网络参数，提升网络性能；
    + 核内剪枝:提升模型性能
![剪枝方式](../img/2019-08-21-15-27-18.png)

    + 最优脑损伤(Optimal brain damage, OBD)[8]：通过移除网络中不重要的连接 ,在网络复杂度和训练误差之间达到一种最优平衡状态 , 极大加快了网络的训练过程.
    + 最优脑手术 (Optimal brain sur-geon, OBS)[9]:损失函数中的 Hessian 矩阵没有约束,这使得 OBS 在其他网络中具有比 OBD 更普遍的泛化能力。上述两种方法都面临者严重的网络精度损失。
    + 深度压缩 (Deep compression)[28]:综合应用了剪枝、量化、编码等方法 , 在不影响精度的前提下可压缩网络 35 ∼ 49 倍 , 使得深度卷积网络移植到移动设备上成为可能。
    + 针对全连接层进行剪枝操作，摆脱了对于训练数据的依赖[29]。
    + 动态网络手术 (Dynamic network surgery)[30]:在剪枝过程中添加了修复操作，可以重新激活重要的操作。交替进行，极大的改变了网络学习效率
    + ReLU 激活函数移至 Winograd域 , 然后对 Winograd 变换之后的权重进行剪枝[31]。
    + LASSO 正则化剔除冗余卷积核与其对应的特征图 , 然后重构剩余网络 , 对于多分支网络也有很好的效果。[32]
    + 去除对于输出精度影响较小的卷积核以及对应的特征图[33]:
    + 一次性 (One-shot) 优化方法:可获得60%∼70%的稀疏度[26]:
    + ThiNet[34]:在训练和预测阶段同时压缩并加速卷积神经网络,从下一卷积层而非当前卷积层的概率信息获取卷积核的重要程度 , 并决定是否剪枝当前卷积核 , 对于紧凑型网络也有不错的压缩效果。
![网络剪枝对不同网络的压缩效果](../img/2019-08-21-15-44-48.png)

## 2. 张量分解

神经网络的主要计算量，来自于卷积层。网络仅仅需要很少一部分参数就可以进行准确的预测[35];卷积核是四维张量。将原始张量分解为若干低秩张量，减少卷积操作数目。

**常见的张量分解方法有：**

- CP分解：![主要公式](../img/2019-08-21-15-56-02.png)
- Tucker分解：将卷积分解为一个核张量与若干因子矩阵。是一种高阶的主成分分析方法。
![主要公式](../img/2019-08-21-15-57-01.png)
- 矩阵奇异值分解 (Singular value decomposition, SVD)常用于全连接层分解:![SVD公式](../img/2019-08-21-15-58-19.png)

![张量分解过程](../img/2019-08-21-15-59-44.png)

图 3 (a)中W为原始张量数据维度为：(d,d,i,o);复杂度为O(d^2*i*o);进行b中的张量分解后复杂度为O(o*k)+O(d^2*k*i);大大降低了复杂度，复杂度降低为原来的(o/k),k越小压缩效果越明显。

张量的典型引用是将高维离散余弦变换(Discrete cosine transform, DCT)分解为一系列一维DCT变换。

卷积的张量分解方法：

- 分离卷积核学习方法 (Learning separable filters)[36]:能够将原始卷积核用低秩卷积核表示,减少所需卷积核数量以降低计算负担。
- 逐层分解方法[37]：每当一个卷积核被分解为若干一阶张量则固定此卷积核并基于一种重构误差标准以微调其余卷积核。在场景文本识别中可加速网络4.5倍
- 全连接层奇异值分解[38]：全连接层进行展开奇异值分解，可以大大减少网可以参数，并提升网络速度。
- 基于cp分解的卷积核张量分解方法[39]：用最小二乘法，将卷积核进行分解为四个一阶卷积核张量。并且表明张量分解具有正则化效果。
- 约束的张量分解新算法[40]：将非凸优化的张量分解转，化为凸优化问题 , 与同类方法相比提速明显。
- 非对称张量分解方法[41]：加速整体网络运行。

**网络整体压缩**

- 基于PCA累积能量的低秩选择方法和具有非先行的重构误差优化方法[41]:
- 基于变分贝叶斯的低秩选择方法和基于 Tucker 张量分解的整体压缩方法[42]：尺寸和运行时间都大大降低。
- 利用循环矩阵剔除特征图中的冗余信息,获取特征图中最本质的特征[43],减少参数但是性能不减少
- 等提出了一种基于优化CP分解全部卷积层的网络压缩方法[44]：克服了由于PC分解带来的网络精度下降问题。

张量分解过后都需要重新训练网络至收敛，进一步加剧了网络训练的复杂度。

## 3. 知识迁移

知识迁移是属于迁移学习的一种网络结构优化方法 , 即将教师网络 (Teacher networks) 的相关领域知识迁移到学生网络 (Student networks)以指导学生网络的训练。

教师网络拥有良好的性能和泛化能力，学生网络拥有更好的针对性和性能

![知识迁移过程](../img/2019-08-21-16-32-50.png)

注释迁移主要由教师网络获取和学生网络训练两部分构成。

- 教师网络：主要要求准确率
- 学生网络:少数数据的快速训练

- 基于知识迁移的模型压缩方法[45]：
- 利用 logits ( 通过 softmax 函数前的输入值 , 均值为 0) 来表示学习到的知识[46]:
- 知识精馏(Knowledge distilling, KD)[47]:采用合适的 T 值 , 可以产生一个类别概率分布较缓和的输出(称为软概率标签(Soft probability labels)).
- FitNet[48]:
- Net2Net[50]:基于函数保留变换 (Function-preserving transfor-mation)可以快速地将教师网络的有用信息迁移到更深 ( 或更宽 ) 的学生网络
- 于注意力的知识迁移方法[51]：从低、中、高三个层次进行注意力迁移;极大改善了残差网络等深度卷积神经网络的性能
- 结合 Fisher 剪枝与知识迁移的优化方法[52]: 利用显著性图训练网络并利用 Fisher 剪枝方法剔除冗余的特征图 , 在图像显著度预测中可加速网络运行多达 10 倍。
- 知识迁移的端到端的多目标检测框架[54]:解决了目标检测任务中存在的欠拟合问题 , 在精度与速度方面都有较大改善。

知识迁移方法能够直接加速网络运行而不需要较高硬件要求，大幅降低了学生网络学习到不重要信息的比例 , 是一种有效的网络结构优化方法 . 然而知识迁移需要研究者确定学生网络的具体结构，对研究者的水平提出了较高的要求。此外,目前的知识迁移方法仅仅将网络输出概率值作为一种领域知识进行迁移，没有考虑到教师网络结构对学生网络结构的影响。提取教师网络的内部结构知识 ( 如神经元 ) 并指导学生网络的训练，有可能使学生网络获得更高的性能。

## 4. 精细模块设计

通过对高效精细化模块的构造，可以实现优化网络结构的目的，采用模块化的网络结构优化方法，网络的设计与构造流程大幅缩短。目前就有代表性的精细模块有：Inception模块、网中网、残差模块

### 4.1 Inception模块

#### 4.1.1 Inception-v1
Szegedy [4] 等从网中网 (Net-work in network, NiN) [55] 中得到启发，提出了如图所示的 Inception-v1 网络结构：

![Inception 结构](../img/2019-08-21-18-51-11.png)

**将不同尺寸的卷积核并行连接能够增加特征提取的多样性**；而引入的 1 × 1 卷积核则加速了网络运行过程。

#### 4.1.2 Inception-v2

因为卷积神经网络在训练时，每层网络的输入分布都会发生改变，会导致模型训练速度降低。因此使用批标准化(Batch normalization BN)。主要用于解决激活函数之前，作用是解决梯度问题[56]。

#### 4.1.3 Inception-v3[57]

除了将 7 × 7 、 5 × 5 等较大的卷积核分解为若干连续的 3 × 3 卷积核 , 还将 n × n 卷积核非对称分解为 1 × n 和 n × 1 两个连续卷积核 ( 当 n = 7 时效果最好 ).

![卷积核分解](../img/2019-08-21-19-00-03.png)

nception 结构与残差结构相结合 , 发现了残差结构可以极大地加快网络的训练速度[58]:

Xception[59]:用卷积核对输入特征图进行卷积操作.

![Xception 模块](../img/2019-08-21-21-12-12.png)

### 4.2 网中网（Network in network）


