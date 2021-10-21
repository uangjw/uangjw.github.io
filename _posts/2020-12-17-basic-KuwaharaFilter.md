---
layout: post
title: 了解Kuwahara Filter
categories: [Basics]
description: 如题
keywords: KuwaharaFilter, Basics
---

​	Kuwahara filter是一种非线性的平滑滤波器。其主要特点是能够在保持边缘的前提下对图像进行平滑处理。

- Kuwahara Operator

  ​	假设$ I(x,y) $是一个灰度图像，Kuwahara filter在其中以像素$ (x,y) $为中心取一个边长为$ 2a+1 $的正方形。这一正方形（window）可以被分成四个子区域（subregion），它们可以被表示为：

  ​	$$Q_i(x,y)=\begin{cases}[x,x+a]\times[y,y+a] \hspace{1em} \rm{if} \ \textit{i}=1 \\[x-a,x]\times[y,y+a] \hspace{1em} \rm{if} \ \textit{i}=2 \\[x-a,x]\times[y-a,y] \hspace{1em} \rm{if} \ \textit{i}=3 \\[x,x+a]\times[y-a,y] \hspace{1em} \rm{if} \ \textit{i}=4\end{cases}$$

  一个区域划分的例子如下图所示，需注意a、b、c、d四个区域是有一定的重叠部分的。

  ​	分别计算四个区域的算术平均值$ m_i(x,y) $与标准差$ \sigma_i(x,y) $，其中标准差最小的区域所具有的算术平均值就成为window中心像素$ (x,y) $的值，也即四个区域中最“均匀”的部分的算数平均值就被取作中心像素的值。记Kuwahara filter的输出值为$ \varPhi (x,y) $，则Kuwahara filter的处理过程可表示为$ \varPhi(x,y)=m_i(x,y) $，其中$ i= \mathop{\arg\min}_{j} \sigma_j(x,y)$。

  ​	像素$ (x,y) $所在位置与边缘的关系对于判断哪一个子区域具有最小标准差来说非常重要。当像素恰落在边缘上时，其所取的值将是来自更平滑的区域的。与中值滤波一样，Kuwahara filter使用一个滑动窗口（sliding window）来处理图像的每一个像素，滑动窗口的大小是事先指定的，其值将影响生成图片的模糊程度。通常取具有奇数边长的正方形作为窗口（实际上也有长方形的窗口，子区域之间也并不非要重叠或一样大，只要能把整个窗口都覆盖住就可以了）。

  <img src="/images/window.png"/>

- 彩色图像

  ​	使用Kuwahara filter处理彩色图像（RGB）时，不能通过对每个通道分别使用Kuwahara filter后再合并在一起。因为对于图像的一个窗口，每个颜色通道中这一窗口对应的子区域的标准差与算术平均值都可能是不同的，比如说R通道中i=1对应的子区域具有最小的标准差，但B通道中i=2对应的子区域具有最小的标准差。因此正确的处理方式应当是，首先将待处理图像转换为HSV图像，再对纯度通道（Value）使用Kuwahara filter。这样就保证了对于一个窗口只有一个确定的子区域能够决定中心像素的值。

- 缺陷

  1. Kuwahara filter没有指出同时有多个子区域具有最小标准差的情况的处理方式。一般来说由于噪音的存在，两个区域具有完全相同标准差的情况是非常罕见的。当两区域具有相近标准差值时，中心像素的值基本上就是由噪声随机决定的（标准差值相近但算术平均值差别非常大），这也就让Kuwahara filter具有了对噪声敏感的缺陷。一种解决方法是取中心像素值为$ \frac{m_1+m_2}{2} $，$ m_1 $与$ m_2 $所对应的子区域之间标准差之差小于某特定值$ D $。
  2. Kuwahara filter容易生成块状的不自然效果（block artifacts），这一现象在纹理丰富的区域尤为明显。其出现是因为窗口的区域划分是方形的，所以一种解决方式是取非矩形（如圆形）的窗口，另一种方法则是根据输入图像来生成特定的窗口。

