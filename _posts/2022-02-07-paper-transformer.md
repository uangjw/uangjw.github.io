---
layout: post
title: 论文笔记||Attention Is All You Need
categories: [PaperNotes]
description: Transformer的基本架构与自注意力机制
keywords: Transformer

---

### Transformer

Music Transformer的模型建立在Transformer模型的基础之上。

论文：Attention Is All You Need

参考博客：https://blog.csdn.net/longxinchen_ml/article/details/86533005

Transformer模型首次于文章Attention Is All You Need提出，如题目所述，模型仅仅依赖注意力机制来表达输入输出之间的全局依赖，完全摒弃了循环与卷积。相对于具有相同性能的其他模型，Transformer更可并行化，且需求的训练时间更短。

#### 模型架构

<img src="/images/transformer1.png" div align=center/>

编码器：6个相同的网络层组成，其中每层都有两个子网络层（或者理解成6个编码器级联）。子层中按传输顺序第一个是一个Multi-Head self-attention机制，即一个自注意力层。第二个子层是一个简单的全连接前馈网络层。

解码器：同样由6个相同的网络层组成，除了与编码器相同的两个子层，解码器中还有一个对编码器的输出执行multi-head attention的子层。

编码器与解码器的层与层之间都采用残差连接，并对每层输出进行归一化（这两步即图中Add&Norm，将层输出与输入相加并归一化，目的是应对梯度消失以及防止过拟合）

#### 自注意力机制

注意力函数（attention function）可以看作是一个从一个查询与一组键值对到一个输出的映射，其中查询、键、值以及输出都是向量，它们的创建都是通过一个权值矩阵与待处理的向量相乘实现的。在NLP情形下，可以认为自注意力机制就是在模型处理输入序列的每个单词时能够关注到整个序列中的所有单词，从而更好地编/解码；具体来说，当我们希望查询某单词在句子的其他位置上的“注意力”，我们就将其他位置上的单词拿来对该单词做“注意力计算（打分）”，计算结果决定了编码该单词的过程中有多重视句子的其它部分。

作者使用的Scaled Dot-Product Attention机制的结构如下图。计算相应的attention值（输出）的流程可以表达为如下公式：
$$
Attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt{d_k}})V
$$
其中$Q$即请求向量，$K$即所有键值组成的向量，$V$即所有键对应值的向量，$d_k$即键值对的维数。这种计算attention的方式是在一般的dot-product attention的基础上多除了一个维数的平方根，这一步可以让梯度更稳定。将注意力分数计算完毕后通过softmax函数来传递结果，这一步使得所有单词的分数都归一化，得到的分数都是正值且和为1。

<img src="/images/transformer3.png" div align=center/>

前面描述模型结构的时候提到实际上用到的是Multi-Head Attention机制（多头注意力机制）。多头注意力机制扩展了模型专注于输入序列不同位置的能力，给出了注意力层的多个“表示子空间”（论文的表述：“能够联合地从不同的表示子空间中关注不同位置的信息”）。结合下图理解多头注意力机制，向量VKQ并行地进行h次线性投影（linear projection，通过矩阵乘法进行），投影结果将被并行地进行scaled dot-product注意力分数计算，然后拼接到一起再被线性投影，得到一个矩阵。公式表达为：
$$
MultiHead(Q,K,V)=Concat(head_1,...,head_h)W^O \\
where \ head_i=Attention(QW_i^Q,KW_I^K,VW_i^V)
$$
<img src="/images/transformer2.png" div align=center/>

<img src="/images/transformer4.png" div align=center/>