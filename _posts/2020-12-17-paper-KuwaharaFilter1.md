---
layout: post
title: 论文笔记||Image and Video Abstraction by Anisotropic Kuwahara Filtering
categories: [PaperNotes]
description: 一种各向异的Kuwahara滤波器
keywords: paper, KuwaharaFilter
---

​	本文提出了一种各向异的Kuwahara滤波器（anisotropic Kuwahara filter），其能够在保持边缘的同时沿局部特征方向来平滑图像，从而生成接近绘画效果的图像。

- 此前的一些经典的保持边缘的图像平滑滤波方法如均值漂移以及双边滤波等，都或多或少存在着一定的缺陷，如在具有高对比度的图像区域，这些滤波方法可能无法产生明显的平滑效果或者平滑过度使得细节丢失严重等等。基础的Kuwahara滤波器相比于前面所提的几个滤波器，在处理高对比度区域以及低对比度区域图像时有一定的优势，但依然具有对噪声不稳定以及会产生块状不自然效果的问题。论文2提出了一种解决此问题的方法，即并不直接取具有最小标准差的子区域来确定中心像素的值，而是根据所有子区域的标准差分配权重，将各子区域的算术平均值的加权和作为结果。显然图像更均匀的子区域所被分配得到的权重将更大。为进一步改进，论文2将滤波器的形状取为了一个圆盘，并按扇形划分子区域。改进后的输出结果大大减少了块状不自然现象的产生，但还是可以观察到一些色彩聚集的不自然，且此种方法产生的绘画效果并没有将原输入图像的方向特征（directional features）考虑进来。本文提出的方法最大的特点是在论文2的基础之上，区别于各向同的（isotropic）滤波方法，其能够根据输入图像的局部特征来调整滤波核的形状与取向（orientation），从而大大减少生成图像中的不自然现象。

- 流程可以分为三步。第一步，计算出输入图像的结构张量并对其进行高斯滤波处理。第二步，从平滑后的结构张量计算得到其特征值以及特征向量，并以此为基础对图像局部的取向与各向异程度进行计算。第三步，使用非线性的滤波器来进行基于输入图像结构的平滑滤波处理。该滤波器使用定义在椭圆上的权重函数，对应着更小标准差的局部平均值具有更大的权重。

  - 取向以及各向异性（anistropy）的估计

    ​    输入图像的局部取向和各向异程度估计是基于结构张量的特征值以及特征向量进行的。本文直接根据输入图像$f$的RGB值计算出了结构张量：

    ​	$$(g_{ij})=\begin{pmatrix}f_x\cdot f_x & f_x\cdot f_y \\f_x \cdot f_y & f_y\cdot f_y\end{pmatrix}=:\begin{pmatrix}E & F \\F & G\end{pmatrix}$$

    其中$f_x$与$f_y$为$f$在x方向以及y方向的偏导，可以通过分别与标准差为$\sigma$的高斯函数的x偏导与y偏导做卷积运算得到。结构张量矩阵$(g_{ij})$的特征值分别对应着变化率平方的最小值与最大值，对应的特征向量则分别指示这两者的方向。将对应最小变化率的特征向量选择出来，可以得到一个向量场（取具有最小变化率的是因为变化率小的区域正是对应着需要被平滑处理的非边缘部分）。未经处理之前此向量场是不具有连续性的，要想得到能够更清晰地反映图像特征的向量场，需要对结构张量先进行平滑处理。本文使用标准差$\sigma=2.0$的高斯滤波器对结构张量进行了线性的平滑处理（对应着具有较大梯度值的边缘将被分配到更大的权值），体现在特征向量上的效果则是高度非线性的，并且对应了几何上的主成分分析。这样处理过后，边缘取向的信息也被分布到了边缘的邻域之中，从而能够在后续的处理中形成影响。

    ​	下面是本文估计局部各向异程度的方法。首先基于已经计算得到的结构张量，可以得到其特征值为

    ​	$$\lambda_{1,2}=\frac{E+G\pm \sqrt{(E-G)^2+4F^2}}{2}$$

    与最小变化率同一方向的特征向量为

    ​	$$t=\begin{pmatrix}\lambda_1-E \\-F\end{pmatrix}$$

    ​	于是可以定义局部取向（local orientation）为$\varphi=\arg t$。各向异程度$A$则可以用下式衡量：

    ​	$$A=\frac{\lambda_1-\lambda_2}{\lambda_1+\lambda_2}$$

    $A$的范围从0到1，当其取0时意味着该局部是各向同的（isotropic，梯度没有一个偏好（preferred）的方向），而取1时意味着该区域是完全各向异的，梯度全部具有同一取向。

  > 结构张量structure tensor：
  >
  > ​	根据结构张量能区分图像的平坦区域、边缘区域和角点区域。给定一张图片$I$，取一任意像素记作$I(p)$，$p$为一个确定的坐标。$I_x$与$I_y$分别是图像在$x$和$y$方向的梯度。给定邻域窗口的半径$m$以及取求和指数坐标$r$，$r$的取值范围为$\{-m,..+m\}\times\{-m,..+m\}$。给定权值矩阵$w$，该权值矩阵的和窗口的尺寸一样，且所有权值元素的和为1。于是每一个像素位置的结构张量就是一个$2\times 2$的矩阵如下所示：
  >
  > ​	$$S_w(p)=\begin{bmatrix} &\sum_r w(r)[I_x(p-r)]^2 & \sum_r w(r)[I_x(p-r)][I_y(p-r)] \\ & \sum_r w(r)[I_x(p-r)][I_y(p-r)] & \sum_r w(r)[I_y(p-r)]^2\end{bmatrix}$$
  >
  > 取窗口的半径$m$为0，那么可以得到对图像中的每一个坐标为$p$的像素，其结构张量矩阵为：
  >
  > ​	$$S(p)=\begin{bmatrix} &I_x(p)^2 & I_x(p)I_y(p) \\ & I_x(p)I_y(p) & I_y(p)^2\end{bmatrix}$$
  >
  > 将该矩阵进行奇异值分解，可以得到两个特征值$\lambda_1$与$\lambda_2$，对应地有特征向量$\vec{e_1}$以及$\vec{e_2}$。若$\lambda_1>\lambda_2$，那么$\vec{e_1}$或$-\vec{e_1}$的方向就是与窗口内梯度最大对齐的方向。实际上$\lambda_1$的取值就是窗口内沿前述方向的梯度的加权和，代表了主方向的强弱程度。同理，$\lambda_2$的取值代表了窗口内次方向的强弱程度。因此，$\lambda_1$与$\lambda_2$的相对差异可以用来衡量窗口中梯度的各向异程度。这里所谓各向异程度（anisotropy）指的是窗口中的梯度在多大程度上偏向某一个特定方向。这一属性可以通过一致性（coherence）来量化衡量。一致性的定义如下：
  >
  > ​	$$c_w=\begin{pmatrix}\frac{\lambda_1-\lambda_2}{\lambda_1+\lambda_2}\end{pmatrix}^2$$

  - 各向异的Kuwahara滤波：

    ​	定义二维到三维的映射$f$为输入图像，二维向量$x_0$为一个点；$\varPhi$为局部取向，$A$为$x_0$处的各向异程度。为使$x_0$处的椭圆形滤波器的离心率（形状）受$A$控制，设

    ​	$$S=\begin{pmatrix}\frac{\alpha}{\alpha+A} & 0 \\0 & \frac{\alpha}{\alpha+A}\end{pmatrix}$$

    其中$\alpha>0$为一个调优参数（tuning parameter）。明显当$\alpha\to\infty$时矩阵$S$趋向于单位矩阵。本文在所有例子中取$\alpha$为1，对应的最大离心率为4。设$R_{\varphi}$为$\varphi$定义的旋转矩阵且设$h=\lceil 2\sigma_r\rceil$。于是该椭圆就可由下面的式子定义：

    ​	$$\Omega=\{x\in R^2:\Vert SR_{-\varphi}x\Vert\leq h\}$$

    这样定义的椭圆窗口的主轴与局部取向对齐（$\varphi$），且其在各向异程度高的区域具有较大离心率，各向异程度低的区域具有较小离心率（趋向圆形）。

    ​	$SR_{\varphi}$定义了一个将椭圆$\Omega$映射到以$h$为半径的圆盘上的线性坐标变换。为了确定椭圆窗口上的滤波权重函数（weighting function），本文通过定义特征函数（characteristic function）来将映射得到的圆盘划分为N个扇区。第i个扇区的定义如下：

    ​	$$\chi _i(x)=\begin{cases}1 \ \ \frac{(2i-1)\pi}{N}<\arg x\leq \frac{(2i+1)\pi}{N}, \ i=0,\dots ,N-1 \\0 \ \ \rm{otherwise}\end{cases}$$

    也即，在扇区i内的点取值为1，在扇区i外的点取值为0。通过特征函数$\chi _i$可定义出扇区上的权值函数：

    ​	$$K_i=(\chi_i\star G_{\sigma_s})\cdot G_{\sigma_r}$$

    特征函数与高斯函数做卷积运算（高斯平滑处理）将使得特征函数之间出现细微的重叠（扇区作为子区域subregion，相互之间有一定的重叠），而平滑结果与另一高斯函数的相乘运算将使结果$K_i$具有随着半径的增加而衰减的性质（靠近圆心的像素具有更大的权值）。由关系$\chi_i=\chi_0\circ R_{-2\pi i/N}$以及高斯滤波函数的中心对称性，$K_i$可以进一步表示为：

    ​	$$K_i=((\chi_0\star G_{\sigma_s})\cdot G_{\sigma_r})\circ R_{-2\pi i/N}=K_0\circ R_{-2\pi i/N}$$

    将$K_i$映射回$\Omega$上，就可以得到椭圆$\Omega$上的滤波权值函数$\omega_i$：

    ​	$$\begin{aligned}\omega_i=&(SR_{-\varphi})^{-1}K_i=K_i\circ SR_{-\varphi}\\=&K_0\circ R_{-2\pi i/N}\end{aligned}$$

    滤波权值函数定义完毕后，可以求出各扇形子区域的加权平均值$m_i$以及方差$s_i^{2}$如下：

    ​	$$m_i=\frac{1}{k}\int f(x)\omega_i(x-x_0)\mathrm{d}x$$

    ​	$$s_i^2=\frac{1}{k}\int f^2(x)\omega_i(x-x_0)\mathrm{d}x-m_i^2$$

    其中$k=\int \omega_i(x-x_0)\mathrm{d}x$，用于归一化。接下来设权值因子$\alpha_i$为：

    ​	$$\alpha_i=\frac{1}{1+\Vert s_i\Vert^q}$$

    于是输出就可定义为：

    ​	$$\frac{\sum_i \alpha_i m_i}{\sum_i\alpha_i}$$

    $\alpha_i$的定义保证了具有更小标准差的扇形子区域会具有更大的权值，同时当$s_i$值为0时其不会处于未定义的状态；其中的$\Vert s_i\Vert ^q$的意义是使子区域的权值受到其自身的标准差的值的影响，因而避免了前面提到过的不确定性；$q$的作用在论文*Artistic Edge and Corner Enhancing Smoothing*中（存在区别，但基本作用一致）有更充分的介绍。

