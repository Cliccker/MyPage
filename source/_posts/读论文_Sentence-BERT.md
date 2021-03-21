---
title: 读论文——Sentence-BERT:基于孪生网络的句子嵌入
author: Hank
mathjax: true
categories: 学习
summary: 一种改进的Bert模型，用以输出句子的Embedding
tags:
  - NLP
  - Bert
  - 问答系统
abbrlink: 947f
date: 2020-08-11 12:47:00
---



原标题：Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks

[论文地址](https://arxiv.org/abs/1908.10084)

[代码](https://github.com/UKPLab/sentence-transformers)

## 背景

`BERT`（2018）和`RoBERTa`（2019）在处理文本语义相似度等句子对回归的任务上，都有不错的效果。但BERT在计算句子相似度时，需要将两个句子都输入到网络中，这就会产生巨大的计算开销。在10000个句子中找到相似的句子需要`5000w`次推理计算，这项任务在V100GPU上大约耗时`65h`。

**BERT的构造使得它既不适用于语义相似度搜索，也不适合于聚类等无监督任务。**

有研究尝试将整个句子输入到BERT中，得到该句的句向量。但是这样得到的句向量包含的语义信息有限，也就是说

**相似的句子的句向量差别可能会很大**

这一点作者也在后续的实验中证明了：

> The results shows that directly using the output of BERT leads to rather poor performances. Averaging the BERT embeddings achieves an average correlation of only 54.81, and using the CLStoken output only achieves an average correlationof 29.19. Both are worse than computing average GloVe embeddings.

## 解决的问题

提出了Sentence-Bert，一种改进的模型。该模型使用孪生网络（Siamese Network）来输出**定长**的、**保有语义信息**的句向量。再通过余弦距离、曼哈顿距离或欧式距离来计算其相似度。SBERT可以将原本需要`65h`的任务缩短至`5s`左右。

## 建模

### 孪生网络（Siamse Network）

<img src="C:\Users\76084\Desktop\20200811192235.png" alt="孪生网络" style="zoom: 33%;" />

孪生网络结构相对简单，在计算句子相似度方面有很多良好的表现。最主要的特征是孪生网络的两个解码器（encoder）共享权重`W`。在输出层，可以通过余弦相似度等方法比较$u,v$两个向量之间的相似度。也可以用额外的模型来生成两个句子之间**关系的特征向量**，用于问答系统等更复杂的任务。

### 使用BERT处理文本匹配任务

<img src="C:\Users\76084\Desktop\20200811200024.png" alt="BERT处理文本匹配任务" style="zoom: 50%;" />

如图，`BERT`的常规做法是将两个句子拼接成一个序列，中间以`SEP`符号进行分隔，经过特定模块进行编码后，取输出层的字向量的`平均值`或第一个特征位置`CLS`作为句子的句向量。这种取平均值或特征值的做法使得`BERT`输出的句向量不包含充分的语义信息。

### Sentence-Bert

SBERT沿用了孪生网络的结构，两个Sentence Encoder使用的是同一个BERT，并在其后加入了一个池化（pooling）操作来实现输出相同大小的句向量。文章一共试验了三种池化策略：

1. CLS-token 以特征位置向量作为句向量
2. MEAN-strategy 求取所有输出向量的平均值（默认）
3. MAX-strategy 求取所有输出向量中的最大值

通过池化操作进一步将BERT输出后的字向量进行特征提取、压缩，得到$u,v$。

最后将$u,v$进行组合，针对不同的任务提出不同的组合方式

#### 针对**分类任务**

<img src="C:\Users\76084\Desktop\20200811200024.png" alt="SBERT的一种训练模型" style="zoom:50%;" />

针对分类任务的训练模型如图1所示。使用元素级差分$|u-v|$来组合$u,v$，将其乘以一个可训练的权重矩阵$W_{t} \in \mathbb{R}^{3 n \times k}$（n为句向量的维度，k为标签的个数）。使用`Softmax`函数进行分类输出

$$
o=\operatorname{softmax}\left(W_{t}(u, v,|u-v|)\right)
$$

其中矩阵的形式如下：

<img src="https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20200813084959.png" alt="矩阵形式" style="zoom:67%;" />

最后使用交叉熵来作为优化，文章中没有提到具体形式，查到其大致形式如下：

$$
L=\frac{1}{N} \sum_{i}^{N} L_{i}=\frac{1}{N} \sum_{i}^{N}\sum_{c=1}^{M} y_{i c} \log \left(p_{i c}\right)
$$
其中：

$M$—— 类别的数量；

$N$—— 样本的数量；

$y_c$—— 指示变量（0或1)，如果该类别和样本的类别相同就是1，否则是0；

$p_c$——对于观测样本属于类别$c$的预测概率。

#### 针对**相似度**计算



<img src="https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20200812131036.png" alt="相似度计算" style="zoom: 50%;" />

直接计算、输出余弦相似度；训练损失函数采取了均方根误差；

$$
\sigma=\sqrt{\frac{\sum d_{i}^{2}}{n}}, \mathrm{i}=1
$$

#### 针对**三元组输入**

<img src="C:\Users\76084\Desktop\20200812184837.png" alt="针对三元组的模型" style="zoom: 33%;" />

给定一个锚定句$a$，正面句$p$，负面句$n$。这个网络的目标是调整三元组中$a$与$p$的距离小于$a$与$n$的距离。其损失函数如下：

$$
\max \left(\left\|s_{a}-s_{p}\right\|-\left\|s_{a}-s_{n}\right\|+\epsilon, 0\right)
$$

$s_x$—— $a/n/p$的句向量；

$||·||$—— 计算距离，如欧式距离、曼哈顿距离；

$\epsilon$ —— 边界，保证$s_p$与$s_a$之间的距离至少要比$s_n$与$s_a$小$\epsilon$，文中取1

## 实验

### 比较组合方式和池化策略的差异

<img src="C:\Users\76084\Desktop\20200812134015.png" alt="不同池化策略和组合方式的差异" style="zoom:67%;" />

可以看到平均池化策略的表现比较好；

再平均池化策略下，使用$(u, v,|u-v|)$组合方式表现比较好。

### 计算效率

<img src="C:\Users\76084\Desktop\20200812134540.png" alt="不同嵌入模型的计算效率比较" style="zoom:67%;" />

## 总结

本文提出了在Siamse Network上改进而来的Sentence-BERT模型，通过其实验结果可以看出，该模型在计算效率上有非常大的提高，可能会在工业领域有比较好的应用。