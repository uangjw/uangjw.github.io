---
layout: post
title: 论文笔记||Artistic Edge and Corner Enhancing Smoothing
categories: [PaperNotes]
description: 一种简单的非线性的边缘保持的局部图像平滑算子
keywords: PaperNotes, KuwaharaFilter
---

​	本文提出了一种简单的非线性的边缘保持的局部图像平滑算子。该算子对Kuwahara滤波器以及以其为代表的一大类基于计算值以及设定的准则进行滤波的滤波器（VCFS, value and criterion filter structure）进行了推广，改善了它们受噪声影响严重、取结果时具有不确定性等问题。

- 在角与边缘保持的平滑算子（ECPS, Edge and Corner Preserving Smoother）之中，Kuwahara 滤波器是一个能够产生类似绘画效果的概念简单的滤波器。但其缺点也很明显（前文已有介绍）。本文所提出的算子主要解决的是Kuwahara滤波器的“当多个子区域同时取得最小标准差值时，输出值不被唯一地确定”这一问题。最原始的Kuwahara滤波器可由以下方程定义：

  ​	$$\begin{aligned}\Phi(x,y)=\sum_im_i(x,y)f_i(x,y) \\ \mathrm{where} \ f_i(x,y)=\begin{cases}1, \ s_i(x,y)=\min_k{s_k(x,y)} \\0, \ \mathrm{otherwise}\end{cases}\end{aligned}$$

  设输入信号为$I(x,y)$，若$I(x,y)$的变化足够平滑，某两个子区域的标准差$s_i$与$s_j$小于噪声级时，相对于噪声，此两者的值就会处于相对大小受噪声影响而频繁变化的状态，反映在滤波上就是滤波的输出结果几乎在随机地选取$s_i$或$s_j$对应子区域的算术平均值，平滑效果不佳。

- 对Kuwahara滤波器的扩展基本可以概括为这样的框架：将特定形状的窗口通过一定分法划为$N$个子区域，每个区域通过滤波权值函数$w_i$运算出子区域内的加权平均值$m_i$以及标准差$s_i$，$i=1\dots N$；设图像为$I$，则有：

  ​	$$\begin{aligned}m_i=I\star w_i, \ s_i^2=I^2\star w_i-m_i^2, \\\mathrm{with}\iint _{R^2}w_i(x,y)\mathrm{d}x\mathrm{d}y\end{aligned}$$

  $\star$表示卷积运算。对于原始的Kuwahara滤波器，当$(x,y)\in Q_i$时，$w_i(x,y)=1$，否则$w_i(x,y)=0$（$i=1\dots 4$）。对于每一个电$(x,y)$，输出值$\Phi(x,y)$由下列结合标准（combination criterion）$\mathcal{C} :R^{2N}\mapsto R$定义：

  ​	$$\Phi(x,y)=\mathcal{C} [m_1(x,y),\dots ,m_N(x,y); \ s_1(x,y),\dots ,s_N(x,y)]$$

- 本文所提出的方法是一个VCFS的推广。其主要提出了一个新的不具有MSDC类滤波方法的限制的结合标准，且使用了一个更适合保持边缘与角的滤波权值函数$w_i$。

  - 灰度图像

    ​	本文所使用的是一个圆形的窗口（所处理的像素的一个圆形邻域）。首先将窗口划分为$N$个相等的扇形，第$i$个扇形用$S_i$表示。设二维高斯滤波核$g_{\sigma}(x,y)$，于是可以使用如下的划分函数（cutting function，在求$V_i$时用到的$U_i$对应着论文1中的characteristic function，与高斯函数相乘后得到随半径增大而衰减的权值）$V_i(r,\vartheta)$来完成扇区的划分：

    ​	$$\begin{aligned}&V_i=U_i\cdot g_{\sigma/4}\\\mathrm{with} \ U_i(r,\vartheta)=&\begin{cases}N, \ i-\frac{1}{2}<\frac{N}{2\pi}\vartheta<i+\frac{1}{2} \\0, \ \mathrm{otherwise}\end{cases}\end{aligned}$$

    其中权值函数$w_i$定义为$w_i=g_{\sigma}\cdot V_i$，于是对任意$x$与$y$有下列关系：

    ​	$$\frac{1}{N}\sum^N_{i=1}V_i(x,y)=1, \ \frac{1}{N}\sum^N_{i=1}w_i(x,y)=g_{\sigma}(x,y)$$

    ​	根据式（20）完成$m_i$与$s_i$的计算以后，引入参数$q$，点$(x,y)$的输出可以定义为：

    ​	$$\Phi_q=\frac{\sum_im_is_i^{-q}}{\sum_is_i^{-q}}$$

    ​	由式（24）可见，输出值$\Phi_q(x,y)$是一个局部平均值$m(x,y)_i$的加权平均值，权值函数为$s_i^{-q}(x,y)$，显然$q>0$时对应$s_i$更小的局部平均值具有更大的权值。

    ​      参数$q$的作用主要在于控制滤波效果从类似高斯滤波的线性滤波到类似VCFS（如最原始的Kuwahara滤波）的转变程度。$q$为0时，$\Phi(x,y)$为$m_i$的算术平均值，由（23）所示关系可知此时本滤波器成为一个线性的高斯滤波器；而$q$趋向于无穷时，$\Phi(x,y)$几乎完全由最小的$s_i$决定，也即本滤波器几乎成为最原始的Kuwahara滤波器。因此，对于一个有限的正的$q$值，本滤波器就能够兼具线性高斯滤波对噪音的平滑能力以及VCFS类型滤波器的边缘保持能力。

    ​	此方法有一个明显的缺陷，就是当$s_i$为0时式（24）是未定义的。尽管此种情况在实际应用时较为少见，但毕竟可能引发错误。论文1的方法便解决了这一问题。

  - 彩色图像

    ​	对于三通道的彩色图像，设各彩色分量为$I^{(1)}(x,y)$，$I^{(2)}(x,y)$，$I^{(3)}(x,y)$，局部平均值为$m_i^{(c)}(x,y)$，局部标准差为$s_i^{(c)}(x,y)$，$c=1,2,3,\ i=1,\dots ,N$。与灰度图像类似，输出$\vec{F}_q$可表示为：

    ​	$$\begin{aligned}&\vec{F}_q=\frac{\sum_i\vec{m}_is_i^{-q}}{\sum_is_i^{-q}},\\\mathrm{with} \ \vec{m}_i&=[m_i^{(1)},m_i^{(2)},m_i^{(3)}]^T, \ s_i=\sqrt{\sum^3_{c=1}[s_i^{(c)}]^2}\end{aligned}$$

    需注意彩色图像的滤波并不是通过将三个色彩分量分别处理再合成来实现的（这种方法的问题在前文已经介绍过）。三个彩色分量所使用的局部平均值是一样的。同时本文作者指出，对于不同的彩色模型，这种方法都能够给出基本一致的滤波结果（除非是信噪比非常低的图像）。

