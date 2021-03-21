---
title: gensim中的Doc2vec模型参数调整
author: Hank
categories: 学习
tags:
  - gensim
  - Doc2vec
  - python
abbrlink: 85dd
date: 2020-08-16 15:28:00
---

最近在用Doc2vec训练模型，现在现在需要做一些参数调整来提高准确率。这边记录一些参数的调整和效果。

```python
Doc2Vec(documents=None, 
        #输入语料库
        corpus_file=None, 
        #LineSentence格式的语料库文件的路径。
        alpha = float,
        #初始学习率
        seed = 5;
        #随机种子
        negative = 5,
        #如果数值大于零，则加入负面采样。数值多大就加入多少个“noise word” 
        dm_mean=None, 
        #当使用DM训练算法时，对上下文向量相加（默认0）；若设为1，则求均值
        dm=1,  
        #默认值为1，表示使用DM模型，否则使用DBOW模型
        dbow_words=0, 
        #当设为1时，则在训练doc_vector（DBOW）的同时训练Word_vector（Skip-gram）；默认为0，          只训练doc_vector，速度更快。
        dm_concat=0, 
        #默认为0，当设为1时，在使用DM训练算法时，直接将上下文向量和Doc向量拼接。
        dm_tag_count=1, 
        #使用dm_concat模式时，每个文档的预期文档标签数。
        sammple = 1e-5,
        #用于配置随机采样哪些高频词的阈值，有用范围是（0，1e-5）。
        min_count = 5,
        #忽略所有词频少于一定数值的单词
        docvecs=None, 
        docvecs_mapfile=None, 
        comment=None, 
        trim_rule=None, 
        #词汇修剪原则，指定某些单词应保留在词汇中
        callbacks=(), 
        #回调列表
        **kwargs)
```

