---
layout: post
title: 论文笔记/复现||python-opencv实现数字绘画重打光（relighting）
categories: [replication, PaperNotes, opencv]
description: 如题
keywords: python, opencv
---

​	Project PaintingLight(<https://github.com/lllyasviel/PaintingLight>) 通过色彩几何（color geometry）的方法实现了对数字绘画进行重打光（relighting）的功能；算法的核心在于定义并测量了图像的笔触密度（stroke density），从而以此为基础生成与给定光源位置对应的打光效果。整个项目可以大致分为定义并测量笔触密度、生成大致光照图像以及生成最终图像三个部分。

1. 定义并测量笔触密度

   ​	作者们发现在数字绘画的创作过程中大致有这样的规律：画家往往会通过用色彩不断“加深”的方式来绘制阴影的效果，即色彩混合程度较高的区域（用原文的话来说是“笔触历史较密集”的区域）通常就是阴影的区域。从这里我们也可以明显地看出，这一出发点将使得本算法对使用大色块绘制（flat）的数字绘画将无可奈何，而只对使用“厚涂”技法的作品能展现出最好的效果；进一步地，“厚涂”作品往往与真实照片接近，因此我们也可以推测本算法在对照片的重打光场景下也有施展的空间。

   ​	作者将笔触密度stroke density定义如下：

   ​	$$k=\frac{1-\sqrt{\sum^n_{i=1}{\alpha^2_i}}}{1-\frac{1}{\sqrt{n}}}$$

   ​	将组成源图像的所有色彩（像素值，一个色彩空间中的向量）的集合称为一个调色盘palette，那么其中的每一个特定的颜色实际上都可以由整个调色盘中的色彩值加权求和得到，上式中的$\alpha_i$就是这一“权值”。通过建立基于凸包的调色盘并利用其几何特性，简单推导后就可得到每一色彩$\boldsymbol{c}_p$与调色盘元素的关系如下：

   ​	$$\begin{aligned}\boldsymbol{c}_p&=(1-\frac{|\boldsymbol{c}_p-\boldsymbol{g}|}{|\boldsymbol{h}_p-\boldsymbol{g}|})\boldsymbol{g}+(\frac{|\boldsymbol{c}_p-\boldsymbol{g}|}{|\boldsymbol{h}_p-\boldsymbol{g}|})\boldsymbol{h}_p\\&=(\frac{\boldsymbol{c}_p-\boldsymbol{g}}{|\boldsymbol{h}_p-\boldsymbol{g}|})\boldsymbol{h}_p+\frac{1}{n}\sum^n_{i=1}{\frac{|\boldsymbol{c}_i-\boldsymbol{g}|}{|\boldsymbol{h}_i-\boldsymbol{g}|}}\end{aligned}$$

   上式中$\boldsymbol{g}$为调色盘的重心$\frac{1}{n}\sum^n_{i=1}{\boldsymbol{c}_i}$，$\boldsymbol{h}$为以$\boldsymbol{g}$为起点，经过特定像素值$\boldsymbol{c}_p$的射线与调色盘对应凸包的交点（坐标）。由上式可以看出，权值$\alpha$即为$\boldsymbol{c}_i$的系数，代入stroke density的定义式后可以得到表达式：

   ​	$$k_p=\frac{|\boldsymbol{c}_p-\boldsymbol{g}|}{|\boldsymbol{h}_p-\boldsymbol{g}|}$$

   于是便可以估计得出源图像的stroke density分布。

2. 生成大致光照图像

   ​	作者注意到画家在绘制光影效果时往往先绘制出一个大致的忽略轮廓与边缘的光影效果，于是设计算法在生成光影效果时也同样地首先生成大致图像。在源代码中，作者使用高斯金字塔与sobel算子搭配来生成能体现微弱轮廓特征的大致的光照效果；此时暂未考虑光源位置的信息。在考虑光源位置对光照效果图像的影响时，作者指出

   > Each peak of the wave function $n_i^*(\boldsymbol{l},\theta,p_d)$ has a side facing the light source and a side facing away from the light source. We can increase the intensity of the front side and reduce the intensity of the back side to simulate a light

   其中$\boldsymbol{l}$为光源位置的坐标，$\theta$与$p_d$是体现光源在图像上的投影坐标与特定像素坐标之间位置关系的变量。这一部分我暂时还是不太理解，看到作者在后文提到其做法类似shape-from-shadow algorithm，我想先了解一下后者，或许能够解开我这里的困惑。总之最终的做法是，将前述wave function的单位方向向量以及光源到待确定像素的单位方向向量做点乘运算，结果就是最终的考虑了光源位置的大致光照图像。

3. 生成最终图像

   ​	在生成最终图像时，根据笔触密度大处倾向于具有“高光”或“阴影”效果的规律，最终的光照效果图像$\boldsymbol{S}$可由下式计算得到：

   ​	$$\boldsymbol{S}_i=\gamma \boldsymbol{E}_i\odot \boldsymbol{K}+O$$

   其中$\boldsymbol{E}$即已考虑了光源位置的大致光照效果图像，$\boldsymbol{K}$为各像素笔触密度stroke density的值所对应的灰度图像，$\gamma$为光照强度参数。$O$为环境强度参数（ambient intensity），其意义是在具有较小笔触密度值的区域模拟环境光的效果。在实际实现中，作者是先生成不考虑光源而考虑笔触密度的光照图像后再将其与用户鼠标指示的光源位置信息（光源坐标向量）作运算，得到最终的光照图像。渲染结果由光照图像与原图像各像素值对应相乘即得。

   ​	大致运行结果如下。代码见<https://github.com/uangjw/Replication-of-PaintingLight> <br>
   <img src="/images/RelightingResult.png"/>
