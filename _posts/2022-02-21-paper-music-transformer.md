---
layout: post
title: 论文笔记||Music Transformer: Generating Music With Long-Term Structure
categories: [PaperNotes, Deep Learning]
description: 利用Transformer与改进的relative positional self-attention生成较长音乐片段
keywords: transformer, music generation

---

### Music Transformer

Music Transformer是一个基于Transformer的音乐生成模型。面对自动音乐生成的问题，Music Transformer在原模型的基础上改进了其所使用的注意力机制；Transformer完全使用注意力层以及全连接层的模型结构可能忽略信号的相对关系特征，Music Transformer在Transformer的基础上使用relative attention，这种注意力机制显式地根据两个标记之间的距离估计注意力分数，从而能够关注在音乐序列中具有更明显作用的相对关系特征。

#### relative positional self-attention

文章Self-Attention with Relative Position Representations首先改进Transformer的注意力机制为relative self-attention。该文指出，Transformer与RNN、CNN不同，不对输入序列的相对或绝对位置信息显式地建模，因此只能把绝对位置的表示也作为输入（即位置编码）来让模型考虑。relative self-attention机制能够有效地考虑相对位置的表示，或者说序列元素之间的距离，从而在进行注意力打分时能够高效地提取出序列基于标记间相对位置的特征。Music Transformer所使用的relative self-attention算法是前述文章的算法的改进版，主要减少了空间复杂度。

Transformer的文章中3.5节就描述了其中使用的位置编码（positional encoding）。在编码器和解码器的输入前，需要给输入的词嵌入向量加上位置编码，从而使得模型能够在学习中利用序列的顺序信息。具体来说，文章提出的模型使用了如下两个函数来保证模型能够轻松地从相对位置信息中学习：
$$
PE_{(pos, 2i)}=sin(pos/10000^{2i/d_{model}}) \\
PE_{(pos, 2i+1)}=cos(pos/10000^{2i/d_{model}})
$$
其中$pos$是token的位置索引，$i$是维度。也就是说，位置编码的每一个维度都是个正弦信号。需要注意到的是，针对任意的偏移常量$k$，都可以用$PE_{pos}$的线性函数表达出$PE_{pos+k}$。

Music Transformer中的注意力机制能够关注到一个序列中两个位置之间相距有多远。注意力层所学习的嵌入$E^r$具有形状$(H,L,D_h)$，其中$H$为多头注意力机制的“头数”，$L$为序列长度，$D_h$为维度。$E^r$中包含的嵌入是，对任意一对位置$i_q,j_k$的相对距离$r=j_k-i_q$，从$i_q$的一个查询向量到$j_k$的一个键向量的嵌入。这些嵌入根据相对距离大小，从$-L+1$排列到0，并且分散地被每一个注意力头所学习。

> 深度学习中embedding“嵌入”到底指什么？NLP中模型的输入是word embedding词嵌入。词嵌入是一种词的类型表示，具有相似意义的词具有相似的表示，是将词汇映射到实数向量的方法总称。词嵌入技术的根本目的是为了方便机器计算。一种非常重要的词嵌入技术是Word2Vec，它是一种用于有效学习从文本语料库嵌入的独立词语的统计方法；核心思想是基于上下文，先用向量代表各个词，然后通过一个预测目标函数学习这些向量的参数。在此技术之后，“似乎一切东西都可以被embedding”。总体来说，embedding一词现在已经指代着这样的技术：构建一个映射将一个空间里的实体抛射到一个线性向量空间里去，这样一来可以在向量空间里计算度量它们的距离，亦或者从这个空间寻找到另一个目标空间的映射关系。

#### self-attention in Transformer

在此先简要总结Transformer中的自注意力机制scaled dot-product attention。

注意力层的输入是$L$个$D$维的向量的集合$X=(x_1,x_2,...,x_L)$。$Q$是查询张量，通过$X$与$D\times D$的权值矩阵$W^Q$相乘得到；键值向量$K$与$V$同理，$KQV$的形状都是$L\times D$。多头注意力机制就是划分前述$KQV$为$H$个$L\times D_h$的张量，$h$为注意力头的索引，维数$D_h=\frac{D}{H}$。“多头”的设计让模型能够关注一个历史token的不同部分。最后注意力打分的计算公式为：
$$
Z^h=Attention(Q^h,K^h,V^h)=Softmax(\frac{Q^hK^{h\top}}{\sqrt{D_h}})V^h
$$

<br>

<img src="images/transformer4.png" div align=center/>

#### relative positional self-attention

文章Self-Attention with Relative Position Representations的注意力机制能够关注到一个序列中两个位置之间相距有多远。注意力层所学习的嵌入$E^r$具有形状$(H,L,D_h)$，其中$H$为多头注意力机制的“头数”，$L$为序列长度，$D_h$为维度。$E^r$中包含的嵌入是，对任意一对位置$i_q,j_k$的相对距离$r=j_k-i_q$，从$i_q$的一个查询向量到$j_k$的一个键向量的嵌入$e_{ij}$。

要理解$e_{ij}$怎么来的，还是需要参考文章Self-Attention with Relative Position Representations的描述。首先这种注意力机制将输入序列建模成一个有向全连接图，两个序列元素间的边用键值向量$a^K_{ij}$与$a^V_{ij}$表示。$a^K_{ij}$用于计算scaled dot-product，即$x_i$到$x_j$的权重系数$e_{ij}$：
$$
e_{ij}=\frac{Q(K+a^K_{ij})\top}{\sqrt{D_h}}
$$
这些嵌入根据相对距离大小，从$-L+1$排列到0，并且分散地被每一个注意力头所学习。$E^r$与查询向量经过一点计算，最后就得到一个$L\times L$的对数矩阵$S^{rel}$。这样，每个“头”的注意力分数就通过下式计算得到（公式中省去了注意力头的索引$h$）：
$$
RelativeAttention = Softmax(\frac{QK^{\top}+S^{rel}}{\sqrt{D_h}})V
$$
Music Transformer使用了相同的方法来在注意力打分过程中引入相对位置信息的影响。Music Transformer优化了$S^{rel}$的计算方法，从而降低了这种注意力机制的空间复杂度。

RPR self-attention的原文章中，从$E^r$每一个注意力头的分量到对应的$S^{rel}$的计算过程包括计算一个$(L,L,D_h)$形状的中间张量$R$，$R$包括了与所有键值之间的相对距离相关联的嵌入（也就是前面的$a^K_{ij}$）。$S^{rel}$的计算是将$L\times D_h$的$Q$拉成$(L,1,D_h)$后与$R$相乘得来的，即$S^{rel}=QR^{\top}$。$R$的存在将算法的空间复杂度拉到了$O(L^2D)$，从而限制了它在长序列中的应用。

Music Transformer中的RPR self-attention做出了这样的改进：

<img src="images/musictransformer.png" div align=center/>

也即，把从$E^r$得到$S^{rel}$的计算过程中对$R$的需求转化成了四个操作，这四个操作的具体效果图片已经展示得很清晰了。
