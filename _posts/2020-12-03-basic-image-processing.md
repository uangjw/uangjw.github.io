---
layout: post
title: "图像处理基础 1"
categories: [Basics]
description: 图像的表示、色彩空间、空间域卷积滤波与频域卷积滤波
keywords: image processing

---

​	图像的表示、色彩空间、空间域卷积滤波与频域卷积滤波。

1. 图像的表示

   - 一幅图像可以被定义维一个二维函数$ f(x,y) $，其中$ (x,y) $是空间（平面）坐标。任何坐标$ (x,y) $的幅度$ f $被定义为图像在这一位置的亮度。图像在$ x $和$ y $坐标以及在幅度变化上是连续的。坐标数字化被称为取样，幅值数字化称为量化。当$ x $，$ y $分量及幅值$ f $都是有限且离散的量时，称图像为数字图像。
   - 取样和量化的结果都是实数矩阵。对图像$ f(x,y) $采样后可得到一幅$M$行$N$列的图像，图像的大小即为$M\times N$。这些离散的坐标都取整数。坐标约定有两种，不同点在于从0还是从1（MATLAB）开始定义坐标。
   - 图像类型
     - 灰度图像：实质时数据矩阵，值表示颜色的“深浅”（从最浅“白色”到最深“黑色”），每个像素只有一个采样颜色。
     - 二值图像：图像上每一个像素只有两种可能的取值或灰度等级状态（也称黑白、单色图像）。
     - 索引图像：把像素值直接作为RGB调色板下标的图像。索引图象可以把像素值“直接映射”为调色板数值，主要用于网络上的图片传呼是和一些对图像像素、大小等有严格要求的地方。索引图象包括一个数据矩阵X，一个颜色映像矩阵Map。其中Map是一个包含三列、若干行的数据阵列，其中每个元素的值均为[0,1]之间的双精度浮点型数据。Map矩阵的每一行分别表示红色、绿色和蓝色的颜色值。（在MATLAB中，索引图象时从像素值到颜色映射表值的“直接映射”。索引表体积小方便传输，只需要把索引表传输过去，接收方用对应的RGB颜色表还原即可）
     - RGB图像：一幅RGB图像是一个$ M\times N\times 3 $的彩色像素数组，其中每个彩色像素是一个三值组。这三个值分别对应一个特定空间位置处该RGB图像的红、绿、蓝分量。RGB也可以视为三幅灰度图像形成的“堆叠”，当将它们送到彩色显示器的红、绿、蓝输入端时，会在屏幕上生成一幅彩色图像。

2. 色彩空间

   ​	色彩空间是使用多种颜色（通常指三种以上）来表示颜色的方法。RGB、HSV、YUV、YCRCB都是色彩空间模型

   - RGB：一幅图像由三个独立的红、蓝、绿通道构成。每个值代表每个像素的每个分量的度量值，值越高也就越“亮”。参数的取值范围都是0~255。
   - 灰度：每个像素值只表示灰度信息这一单一信息。
   - HSV：HSV颜色空间是面向色度的颜色坐标系统的一种，是针对用户视觉感受的一种彩色模型。HSV对应于画家配色的方法。画家通常用改变瑟农和色深的方法从某种纯色中获得具有不同色调的颜色，如在一种纯色中加入白色可以改变色浓，加入黑色可以改变色深，同时加入不同比例的白色与黑色即可获得各种不同的色调。HSV的三个通道表示色度（H即Hue，色调、色相，给出颜色光谱构成的一种度量），饱和度（S即Saturation，给出主波长中的纯光比例，表明一种颜色距离相同亮度灰度的程度），和纯度（V即Value，给出相对于白色光照强度的亮度），对应于直觉上的色影、明暗与色调。HSV广泛用于色彩的比较，在HSV颜色空间下，比RGB更容易跟踪某种颜色的物体，常用于分割指定颜色的物体。（OpenCV中三个分量的范围H=[0,179]，S=[0,255]，V=[0,255]）
   - YCrCb：该空间广泛用于视频和图像压缩，不能算作纯粹的色彩空间。它是RGB颜色空间的一种解码方式。Y通道表示亮度，而Cr和Cb表示红色差值(在RGB空间中R通道和Y的差值）和蓝色差值(在RGB空间中B通道和Y的差值）各自的色度分量。由于人眼对图像的亮度变化更敏感，因此可以通过对色度分量进行降采样来达到图像数据压缩的目的。

3. 滤波方法

   - 空间域卷积滤波

     - 在数字图像处理过程中，空间域是指由像素组成的空间，也就是图像域。空间域方法是指直接作用于图像像素值并改变其特性的处理方法。

     - 二维图像的卷积运算与相关运算：概括来说，相关是将滤波器在图像上滑动，对应位置相乘求和；卷积则先将滤波器旋转180度（行列均对称翻转），然后使用旋转后的滤波器使用相关运算。两者在计算方式上可以等价。（另一种说法：在实际进行计算的时候，都是用翻转以后的矩阵，直接求矩阵内积（加权求和）就可以了（不用区分相关和卷积））。称与每个像素值进行相乘的来自卷积核的值为卷积权重。需注意卷积的区域不一定是矩形，对应地，卷积核也不一定是矩阵的形式。

     - 例子：空间域的图像平滑滤波

       ​	空间域的图像平滑滤波是利用空域滤波器对图像中每个像素的邻域信息进行滤波操作而达到平滑的目的。空域平滑滤波方法可以表示为

       ​	$$g(x,y)=\sum^a_{s=-a}\sum^b_{t=-b}w(s,t)f(x+s,y+t)$$

       其中$ f(x,y) $表示原图像中坐标为$ (x,y) $的待处理像素的灰度值。$ g(x,y) $表示滤波后对应像素的灰度值。$ w(s,t) $表示平滑滤波器，也称为模板或掩模。$a$和$b$表示邻域的范围。如果$a$和$b$都取1，则表示利用$3\times 3$的平滑滤波器来对图像进行操作。根据上式进行运算（$ f $与$ g $进行卷积运算）就可得到$ g(x,y) $的值。然后对原图像中的每个像素进行相同的滤波操作，就可以得到空域平滑滤波后的图像。

   - 频域卷积滤波

     - 根据信息（包括信号和噪声）在空间域和频率域的对应关系，可以判断随空间位置强度突变的信息在频率域是高频，而强度缓慢变化的信息在频率域是低频。具体到图像，边缘和噪声对应高频区域，而背景或灰度变化缓慢的区域则对应低频区域。频率强度与空域上的像素灰度变化特性之间的关系可以用“低频部分反应图像的概貌，高频部分反应图像的细节”来总结。因此可以使用高通滤波器来对图像进行锐化处理。

     - 边缘检测：

       ​	边缘检测就是确定图像中某个局部区域是否存在边缘点，以及若存在还需要进一步确定其位置的过程。两个具有不同灰度值的相邻区域之间总存在着边缘。图像边缘有方向和幅度两个特性。一般沿着边缘走向的灰度值变化缓慢，而垂直于边缘走向的灰度值则变化骄傲的。边缘处灰度变化形式的不同形成了不同类型的边缘。常见的边缘类型包括阶跃型（灰度突变式和渐变式）、脉冲型和屋顶型。对于阶跃型边缘，其灰度值剖面的一阶导数在图像由暗变亮的位置处总有一个向上的阶跃，而在其他位置都为0。这表明可以利用灰度值一阶导数的幅度值来检测图像中局部区域内是否存在边缘。

     - 一阶导数算子：

       ​	在图像处理过程中，一阶导数是通过图像梯度（gradient）来实现的。因此，利用一阶导数检测图像边缘的方法也称为梯度算子法。对于连续函数$ f(a,b) $，它在$ (a,b) $处的梯度可表示为一个二维向量：

       ​		$$\bigtriangledown f(a,b)=[G_a,G_b]^T=[\frac{\partial f}{\partial a},\frac{\partial f}{\partial b}]^T$$

       这个向量的幅度值和方向角分别为

       ​		$$\Vert \bigtriangledown f\Vert=(G_a^2+G_b^2)^{\frac{1}{2}}$$

       ​		$$\varphi (a,b)=\arctan(G_b/G_a)$$

       其中，梯度的幅度值表示边缘的强度；梯度的方向与边缘的走向垂直。

       ​	需要注意的是，在实践中，由于噪声等因素的存在，梯度方向通常不能正确表示边缘的方向，边缘的方向一般需要通过方向匹配模板法等方法来获得。因此，梯度的幅值一般就简称为梯度。此外，为了避免平方和开方运算，减少计算量，以下两种计算梯度的方法也是常用的：

       ​	$$\Vert \bigtriangledown f\Vert=|G_a|+|G_b|$$

       ​	$$\Vert \bigtriangledown f\Vert=\max\{|G_a|,|G_b|\}$$

       ​	由于数字图像中的数据是离散的，幅值有限，其灰度发生变化的最短距离是在两个相邻像素之间，因此在数字图像处理过程中，常用差分代替导数。而连续函数$ f(a,b) $的梯度在$ a $和$ b $方向的分量对应于数字图像$ f(x,y) $在水平和垂直方向的差分。所以，水平和垂直方向的梯度可定义为：

       ​	$$\begin{cases}\begin{aligned}G_h(x,y)&=f(x,y)-f(x,y-1) \\G_v(x,y)&=f(x,y)-f(x-1,y)\end{aligned}\end{cases}$$

       在实际运用时各种导数算子常用小区域模板表示。利用梯度模板对图像进行处理以检测图像边缘实际上就是利用模板与图像进行卷积运算求得水平与垂直方向的梯度，进而求出图像的梯度。实际应用中，可以根据不同的图像选择不同的梯度公式，所得结果均称为梯度图像。而对梯度图像再进行阈值分割（如将大于阈值的梯度设置成白色，小于阈值的梯度设置为黑色），就可以将图像中的边缘提取出来。

     - Prewitt算子（Prewitt Operator）：

       ​	梯度算子类边缘检测方法的效果类似于高通滤波，有增强高频分量，抑制低频分量的作用。Prewitt算子利用像素点上下、左右邻点的平均灰度差分来计算梯度。由于该算子中引入了类似局部平均的运算方式，因此对噪声具有平滑作用，能在一定程度上消除噪声的影响并去掉部分伪边缘。

       ​	Prewitt算子对应的水平和垂直方向模板为：

       ​	$$H_h= \left[\begin{array}{ccc}-1 & 0 & 1\\-1 & 0 & 1\\-1 & 0 & 1\end{array}\right],\ H_v=\left[\begin{array}{ccc}-1 & -1 & -1\\0 & 0 & 0\\1 & 1 & 1\end{array}\right]$$

     - Sobel算子（Sobel Operator）：

       ​	Sobel算子可以看作是在Prewitt算子的基础上，将Prewitt算子中的平均差分改为加权差分而形成的。Sobel算子和Prewitt算子类似，都在检测边缘点的同时具有抑制噪声的能力。但与Prewitt算子相比，Sobel算子认为邻域的像素对当前像素产生的影响不是等价的，所以距离不同的像素对运算结果产生的影响不同，因而具有不同的权值。一般来说，距离越大，产生的影响越小。Sobel算子的两个模板分别对垂直边缘和水平边缘响应最大，即把重点放在接近于模板中心的像素点。因此，Sobel算子检测到边缘的模糊程度略低于Prewitt算子。

       ​	Sobel算子对应的水平和垂直方向模板为：

       $$H_h= \left[\begin{array}{ccc}-1 & 0 & 1\\-2 & 0 & 2\\-1 & 0 & 1\end{array}\right],\H_v=\left[\begin{array}{ccc}-1 & -2 & -1\\0 & 0 & 0\\1 & 2 & 1\end{array}\right]$$
