---
title: Games202-08工业界的实时渲染
tags: 
- CG
categories: 
- - CG 
  - GAMES202
- - Theory
date: 2023-01-25 17:28:10
---
# Temporal Anti-Aliasing（TAA）

最早temporal的思路是用来解决Anti-Aliasing的，先有TAA的巨大成功才会有RTRT里的应用。

> 参考：[Temporal Anti-Aliasing - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/20786650)

## 为什么出现走样？

回顾101学习的关于采样的过程：

![[https://www.researchgate.net/figure/The-evolution-of-sampling-theorem-a-The-time-domain-of-the-band-limited-signal-and-b_fig5_301556095](https://www.researchgate.net/figure/The-evolution-of-sampling-theorem-a-The-time-domain-of-the-band-limited-signal-and-b_fig5_301556095)](Games202工业界的实时渲染/Untitled.png)


（a）是时域上的某个信号，到了频域上（b）是一个连续的频谱（Tips：频谱是左右对称的，即使不存在负的频率）

（c）是冲激函数，只在特定时域上有值，反映到频域上是（d）

（e）是冲激函数对（a）进行采样（也就是二者相乘），得到离散的点，反映到频域上是（f）

**采样就等价于把信号在频域上以采样信号的周期进行延拓**。

![](Games202工业界的实时渲染/Untitled%201.png)

**走样原理**：之所以会出现走样现象是因为采样比较稀疏，也就是采样点之间的间隔比较大，就意味着频谱之间的间隔小，那么延拓之后会产生频谱重叠的现象从而发生走样。

下面是相关参考资料：

> 关于采样的通俗理解：[https://www.zhihu.com/question/431920644/answer/1597049082](https://www.zhihu.com/question/431920644/answer/1597049082)
> 

> 关于傅里叶变换的通俗理解：[https://zhuanlan.zhihu.com/p/19763358](https://zhuanlan.zhihu.com/p/19763358)
> 

> **采样 == 重复的搬移频谱：**[https://blog.csdn.net/qq_38972634/article/details/119832068](https://blog.csdn.net/qq_38972634/article/details/119832068)
> 

## TAA反走样

对于走样的终极解决方案就是：用更多的样本。常用的反走样方法有：SSAA、MSAA、TAA。

TAA的思路也是需要用更多的sample，只不过是当前帧会复用上一帧的sample，使得这一帧仍然用1SPP，但是无形中通过时间上的复用，增加了SPP（sample per pixel），其思路与RTRT中如何运用temporal的思路完全相同。

Temporal AA尝试用在避免性能损失的情况近似Super Sampling AA的结果。它的做法一句话总结就是，把样本分布到过去的N帧中去，然后每一帧从过去的N帧中取得样本信息然后Filter，达到N倍Super Sampling的效果，如下图：

![[Temporal Anti-Aliasing - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/20786650)](Games202工业界的实时渲染/Untitled%202.png)

### **静止场景**

通过TAA提供的一个叫Jittered sampling的方法去复用上一帧的sample，认为连续的四帧之间有一个移动的pattern，它们在时间上各不相同，样本就这样随时间从左上，左下，右上，右下交替采样。

由于在时间上复用，到了frame3的时候，采样的是右下角的点，其实它已经复用了前面4帧的4个点（frame-1，frame0，frame1，frame2），那么到了当前帧继续复用，于是就把temporal的这些样本都考虑进去了，由于场景静止，该操作相当于把之前各自样本的结果在一起做了一个加权平均，这样就等价于在当前帧中做了一个2*2的upsampling：

![](Games202工业界的实时渲染/Untitled%203.png)

之所以不随机采样点是因为**随机的效果不一定好**。在temporal中可能会引入额外的高频的信息，因此使用规定好的样本点，如上图每四帧一个，这样固定样本点位置避免了分布不均匀的情况。

### 运动场景

原本在静止情况下我们是在一个像素里找sample的结果并复用上一帧中的样本点的结果，在运动场景中，当前帧几何的位置通过motion vector找到上一帧中对应的位置，并复用其结果，如果temporal的信息不太可信时，使用clamping方法，也就是把上一帧的结果拉到接近当前帧的结果。

## 拓展：对比 MSAA 与 SSAA

### SSAA

SSAA可以看作是将一个场景按照几倍的分辨率先渲染后再降采样，把几个像素的结果平均起来，思路是完全正确的，但开销较大，因此帧率会严格按照倍数下降。

### MSAA

MSAA则是在SSAA的基础上做了一个近似从而使得其效率提升开销没那么大。

**核心思想：MSAA同样对于每个像素进行了4次子采样（Sample），但是只在像素中心位置运行一次像素着色。**

**尤其注意：“只在像素中心位置运行一次像素着色”指的是对于同一个primitive（几何体），如果一个像素感知到多个primitive，就进行多次像素着色，原因在于子采样点的深度会随着多个不同的primitive而改变，最终呈现出来的像素颜色是最后一个primitive进行的着色。**

![](Games202工业界的实时渲染/Untitled%204.png)

为了实现这样的操作，MSAA内部会对每个像素维护一张表，记录每个子采样点的**颜色值和深度值。**

上图为例，在前向渲染中，三角形的绘制是依次进行的。绘制粉色三角形时，MSAA的具体执行步骤如下：

1. 光栅化阶段，对四个位置的Sample执行三角形覆盖判断，并把深度写入表中。
2. 像素着色阶段，在像素中心圆点处执行像素着色器。该点的位置、深度、法线、纹理坐标等信息由三角形三个顶点重心插值得到。
3. 对四个Sample执行模板测试与深度测试，并将测试通过的Sample数据写入四倍分辨率的模板缓冲与深度缓冲。每个Sample都拥有自己的深度值，依然是重心插值得到。
4. 上图中0、3、2号Sample通过了深度测试，将粉色复制到这3个Sample对应的颜色缓冲中剩下的1个Sample暂为背景色。
5. 上述的子采样点操作结束后，对最终像素颜色进行变更，位置是该像素中心点，求四个Sample的颜色的均值（或者插值）作为最终像素颜色。
6. 重复上述流程绘制第二个蓝色三角形，将像素着色获得的蓝色复制到1号Sample中。
7. 上述的子采样点操作结束后，继续对最终像素颜色进行变更，位置依旧是该像素中心点，求四个Sample的颜色的均值（或者插值）作为最终像素颜色。

可以看到在MSAA流程中所使用的所有缓冲区都变成了原来的四倍大小，这也是为什么MSAA增加了非常多的显存和带宽消耗。

由此可见，上图一个像素有四个sample，感知到两个primitive，只做两次shading：

- 第一次，粉色三角形覆盖到了0，3，2号样本点，更新缓冲区，在像素中心做shading。
- 第二次，蓝色三角形覆盖到了1号样本点，更新缓冲区，在像素中心做shading。

另外，MSAA还允许进行空间上的sample reuse：

![[https://www.sapphirenation.net/](https://www.sapphirenation.net/)](Games202工业界的实时渲染/Untitled%205.png)

如图，在1和2两个像素内，在两个像素的连接处有两个采样点，这两个采样点既可以贡献给像素1也可以贡献给像素2，因此实际上等于通过reuse在6个采样点的情况下得到了8个采样点的结果，减少了采样点的数量，提升了效率。

## 拓展：基于图像的反走样

### 简介

基于图像的反走样（image based anti-aliasing solution）的思路是先渲染出有锯齿的图，然后通过图像处理的方法将锯齿给提取出来并替换成没有锯齿的图。目前最流行的方法叫

基于图像的反走样的方法从FXAA发展而来：FXAA → MLAA → SMAA

### 算法介绍

**FXAA**

FXAA（Fast Approximate Anti-Aliasing）：**快速近似抗锯齿**，是一种典型的边缘检查取样操作，通过提取像素界面周围的颜色信息并完成混合来消除高对比度界面所产生的锯齿。因此FXAA算法的核心思路就两点：

1. **边缘判定算法**
2. **边缘像素的混合因子计算**

具体学习：[FXAA算法演义 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/373379681)

**MLAA**

MLAA (Morphological Anti-Aliasing) ：形态抗锯齿，是对FXAA的改进（应该是增加了矢量化的方法）。

![](Games202工业界的实时渲染/Untitled%206.png)

MLAA处理的流程：

1. 边缘检测， 得到每个像素的边缘，也就是锯齿边界；
2. 沿着锯齿边界，向两侧搜索锯齿边界的终点，也就是锯齿边界结束的位置；
3. 根据两侧锯齿边界结束的位置，将像素矢量化，作出一条蓝色的线，作为重矢量化线；
4. 算出重矢量化线覆盖像素的面积，作为像素间的混合系数；
5. 根据混合系数对像素进行混合。

具体学习：[主流抗锯齿方案详解（四）SMAA - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/342211163)

**SMAA**

SMAA（Enhanced subpixel morphological Anti-Aliasing）：增强的亚像素形态抗锯齿 ，是对MLAA的改进，进行了更准确的边界判断等优化，并且结合了MLAA与超采样的策略（MSAA、SSAA）。·

具体学习：[主流抗锯齿方案详解（四）SMAA - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/342211163)

## 拓展：关于G－Buffer

**G-Buffer的底层结构**

G-Buffer，全称Geometric Buffer ，译作几何缓冲区，本质就算一个帧缓冲对象。它主要用于存储每个像素对应的**位置（Position），法线（Normal），漫反射颜色（Diffuse Color）以及其他有用材质参数**。根据这些信息，就可以在像空间（二维空间）中对每个像素进行光照处理。

![](Games202工业界的实时渲染/Untitled%207.png)

要注意一点：**G-buffers是绝对不能做反走样的**！因为这是有意义的数值，如果进行混合，数值就失去了原本的意义，无法对混合后的数值进行解释。

# 时域超分辨率 Temporal Super Resolution

超分辨率：Super resolution (or super sampling) 就是把低分辨率变成高分辨率。DLSS就是这么一种技术，将一张低分辨率的图输入最后得到一张高分辨率的输出。

## DLSS

DLSS（Deep Learning Super Sampling）：深度学习超采样。

### DLSS 1.0

**思路**

通过数据驱动的方法来做，针对于每个游戏或者场景单独训练出一个神经网络，去学习一些常见的物体边缘，从而在低分辨率拉成高分辨率之后将模糊的边缘换成不模糊的边缘。

**特点**

只在当前帧中进行，不依靠temporal的累积，等于没有任何的额外信息来源。但我们将低分辨率硬拉成高分辨率，如果不想让最后的结果模糊，必须需要一些额外的信息，DLSS1.0是**通过猜测**来提供额外信息的。

### DLSS 2.0

**思路**

DLSS 2.0则摒弃了通过神经网络猜测的结果，而是更希望去利用temporal的信息，因此其核心思路在于TAA，更多的去结合上一帧的信息运用到当前帧中，仍然是temporal reuse。对于静止场景中，连续四帧我们使用不同的感知sample点，当前帧使用上一帧的信息就等于是变相的提升了分辨率。

**存在的问题**

- 如果有temporal failure时，不能使用clamping的方法来解决。原因在于：
    
    ![](Games202工业界的实时渲染/Untitled%208.png)
    
    - 最终要的是一个增大了分辨率的图，分辨率提高也就是像素点增多，那么我们需要知道新增加的小的pixel的像素值是多少。如果此时我们用上一帧的结果盲目的clamping势必会因为一些小的像素的值是根据周围的点的颜色猜测出来的，而且猜测的值很像周围的点，也就是得到了一个高分辨率的图但是很模糊。
    - 总结来说由于DLSS真正的提升了分辨率，因此我们要求新产生的像素的值是要与之前有本质的不同的，否则就会得到一个糊掉的结果。

**解决**

因此我们需要一个比clamping更好的复用temporal信息的方案。

![](Games202工业界的实时渲染/Untitled%209.png)

左边中的蓝色代表上一帧，绿色代表当前帧。

绿点是当前帧给了一个采样信号得到的值，在上一帧也就是蓝色曲线中我们可以从另一个信号采样出来值，最后我们要把二者综合在一起得出一个当前帧增加了采样点后的值。DLSS2.0中的神经网络没有输出任何混合后的颜色，而是去告诉你应该怎么将上一帧找到的信息和当前帧结合在一起。

**效果**

![](Games202工业界的实时渲染/Untitled%2010.png)

![](Games202工业界的实时渲染/Untitled%2011.png)

![](Games202工业界的实时渲染/Untitled%2012.png)

**工业方面**

如果DLSS每一帧需要消耗30ms，那DLSS就太慢了，因此训练出这个网络之后去提升inference性能，针对Nvidia的硬件进行优化，但具体如何做的就是Nvidia内部的事情了。

**其他公司的“DLSS”算法**

- By AMD：FidelityFX Super Resolution
- By Facebook：Neural Supersampling for Real-time Rendering [Xiao et al.]

# 延迟渲染 Deferred Shading

## 为何出现延迟渲染？Why

**目标**

Deferred Shading是一个节省shading时间的方法，是为了让shading变得更高效更快。

**传统光栅化过程**

$$
Triangles → fragments → depth test → shading → pixel
$$

这可能会出现每一个fragment都需要shading的情况，比如需要从远处到近处渲染时，需要将每一个fragment都进行shading，因为每个fragment都会通过深度测试，但最后呈现到图像上只有最后一个fragment的着色结果。

此时的复杂度为：$O(fragment * light)$ ，因为每一个fragment针对每一个light都要计算着色。

**关键**

很多fragment在渲染过程中会通过depth test，但最终可能会因为被后续的fragment所覆盖而不会被看到，因此这些在中间过程中被shading的fragment浪费了很多的资源。

**解决思路**

只去渲染可见的fragment，这就是延迟渲染出现的原因。

## 什么是延迟渲染？What

延迟渲染( Deferred Rendering) ，即延迟着色（Deferred Shading），是将着色计算延迟到深度测试之后进行处理的一种渲染方法。

可以将延迟渲染( Deferred Rendering) 理解为先将所有物体都先绘制到 **屏幕空间的缓冲（即G-buffer，Geometric Buffer，几何缓冲区）** 中，再逐光源对该缓冲进行着色的过程，从而避免了因计算被深度测试丢弃的片元的着色而产生的不必要的开销。也就是说延迟渲染基本思想是，先执行深度测试，再进行着色计算，将本来在物空 间（三维空间）进行光照计算放到了像空间（二维空间）进行处理。

其本质是通过将几何通道与光照通道分离，能够以比标准多通道前向渲染器更低的成本渲染更多的灯光。

## 如何进行延迟渲染？How

可以将延迟渲染理解为两个Pass的过程：

- Pass1（几何通道的光栅化，Geometry Pass）：在第一次光栅化中得到fragment之后我们不做shading，所有的fragment只对深度缓存（depth buffer）做一个更新。在更新深度缓存的同时获取最前深度对象的各种几何信息，并存储到多个G-buffer中（要开启MRT，一次性将不同数据存储到不同的缓冲区）；
- Pass2（光照通道的光栅化，Light Pass）：由于在Pass1中我们已经写好了depth buffer并且知道了最前深度，因此在pass2中只有深度等于最浅深度的fragment才可以通过depth test并进行shading，从而实现了只对visible fragment着色。光照计算的过程还是和正向渲染一样，只是现在我们需要从对应的G-buffer而不是顶点着色器(和一些uniform变量)那里获取输入变量了。

**隐含的信息**：光栅化场景的开销小于对所有fragment着色的开销。

### **前向渲染 vs 延迟渲染**

**前向渲染**（对于每一个需要渲染的物体，都要逐光源渲染，每个光源都会带来一定的渲染成本）

- 正向渲染（Forward Rendering），先执行着色计算，再执行深度测试。
- 正向渲染渲染g个物体，物体着色的片元数量为f，在l个光源下的着色，复杂度为$O(g\cdot f\cdot l)$。
- Forward Rendering，光源数量对计算复杂度影响巨大，所以比较适合户外这种光源较少的场景。

**延迟渲染**（将计算量非常大的渲染(如光照)延时到后期进行处理）

- 延迟渲染( Deferred Rendering)，先执行深度测试，再执行着色计算。
- 由于最耗时的光照计算延迟到后处理阶段，所以跟场景的物体数量解耦，只跟Render Targe尺寸相关，复杂度是$O(N_{light} \times Width_{RT} \times Height_{RT})$。
- Deferred Rendering 的最大的优势就是将光源的数目和场景中物体的数目在复杂度层面上完全分开。也就是说场景中不管是一个三角形还是一百万个三角形，最后的着色复杂度不会随光源数目变化而产生巨大变化。

|  | 前向渲染 | 延迟渲染 |
| --- | --- | --- |
| 场景复杂度 | 简单 | 复杂 |
| 光源支持数量 | 少 | 多 |
| 时间复杂度 | $O(g\cdot f\cdot l)$ | $O(N_{light} \times Width_{RT} \times Height_{RT})$ |
| 抗锯齿 | MSAA, FXAA, SMAA | TAA, SMAA |
| 显存和带宽 | 较低 | 较高 |
| Pass数量 | 少 | 多 |
| MRT | 无要求 | 要求 |
| 过绘制 | 严重 | 良好避免 |
| 材质类型 | 多 | 少 |
| 画质 | 清晰，抗锯齿效果好 | 模糊，抗锯齿效果打折扣 |
| 透明物体 | 不透明物体、Masked物体、半透明物体 | 不透明物体、Masked物体，不支持半透明物体 |
| 屏幕分辨率 | 低 | 高 |
| 硬件要求 | 低，几乎覆盖100%的硬件设备 | 较高，需MRT的支持，需要Shader Model 3.0+ |
| 实现复杂度 | 低 | 高 |
| 后处理 | 无法支持需要法线和深度等信息的后处理 | 支持需要法线和深度等信息的后处理，如SSAO、SSR、SSGI等 |

### 延迟渲染的缺点

- 内存开销较大，需要缓存的信息（G-buffer）很多
- 读写G-buffer的内存带宽用量是性能瓶颈
- 对透明物体：所有半透明的物体都需要等待不透明物体以延迟渲染完成之后，在用前向渲染的方式渲染半透明物体。
- 延迟渲染不可以使用MSAA
    - MSAA本质上是一种发生在光栅化阶段的技术，也就是几何阶段后，着色阶段前，用这个技术需要用到场景中的几何信息（交界处的几何体的深度值）
    - 延迟渲染因为需要节省光照计算的原因，事先把所有信息都放在了G-buffer上，着色计算的时候已经丢失了几何信息
    - 解决方法：可以通过TAA或者是imaged based AA来解决；或者参考如何在延迟渲染中进行MSAA（[《Multisample anti-aliasing in deferred rendering》](https://diglib.eg.org/bitstream/handle/10.2312/egs20201008/021-024.pdf?sequence=1&isAllowed=y)），它的核心思想在于Geometry Pass除了存储标准的GBuffer，还额外增加了GBuffer对应的扩展数据，这些扩展数据仅仅包含了需要多采样像素的MSAA数据，以供Lighting Pass着色时使用。这样既能满足MSAA的要求，又可以降低GBuffer的占用。

到此，对fragment已经优化了很多，而为了更优方法，就将目光放在了light身上，从而引出Tiled Shading。

## Tile-Based Deferred Rendering（TBDR）

TBDR是建立在Deferred Shading的基础之上，将屏幕分成若干个小块，比如一个小块是32 * 32，然后对每个小块单独的做shading。

![](Games202工业界的实时渲染/Untitled%2013.png)

上图是从camera（下方）看向屏幕（上方）的俯视平面图，将屏幕分成了若干个小块，圆代表的是光源的范围，每个圆代表一个光源。上图中的数字代表该区域内会感知到的光源数量。

**核心思想**：节省每个小块要考虑的light数量，每个切出来的小块代表场景中3D的区域，并不是所有的光源都会跟这片区域相交。

在做light sampling时知道光照强度会随着距离的平方衰减，因此面光源或点光源的覆盖范围是很小的，因此可以设定一个范围，把光源的覆盖范围看做一个球形。在渲染时，只需要找该区域能够感知到的光源即可，不需要考虑所有的光源。

**复杂度**：从$O(N_{light} \times Width_{RT} \times Height_{RT})$提升到了 $O(N_{average-light-per-tile} \times Width_{RT} \times Height_{RT})$，即光源数量从总光源数降低为每小块区域的平均光源数量。

但在此之后，又有人对其进行了一个复杂的优化。

## ****Clustered Deferred Rendering****

在刚才的基础上，不仅将其分成若干个小块，还要在深度对其进行切片，也就是将整个3D空间拆分成了若干个网格。

![](Games202工业界的实时渲染/Untitled%2014.png)

**核心思想**：如果只分成若干个小块区域的话，区域内的深度可能会十分大，因此光源可能会对这个小块区域有贡献，但不一定会对根据深度细分后的网格有贡献。所以再划分过后我们可以发现每个网格内的光源数量更少了。

**复杂度**：从$O(N_{average-light-per-tile} \times Width_{RT} \times Height_{RT})$提升到了 $O(N_{average-light-per-cluster} \times Width_{RT} \times Height_{RT})$

以上两种改进的共同点：**避免没有意义的shading从而提高效率**。

# 层次细节 Level of Detail Solutions

LOD的核心思路：生成不同的level，在任何计算过程中能够快速准确的找出一个正确的level去进行各种运算。在工业界中运用LOD方法或者在不同情况下选用不同层级的思路称之为”**Cascaded方法**“。

例如texture的mip-map就是一个LOD，在high level时detail少一点，但可以很快的计算出一个区域的平均。

### Cascaded shadow maps

当shadow map上的一个texel离camera越近，其在屏幕中所覆盖的内容越多，反之离camera越远，其所覆盖的内容越少，因此我们可以知道从camera出发看向场景，离camera远的我们可以用粗糙的shadow map。

但我们是无法拥有一个会变化的shadow map的，因此在实际操作过程中，通常会生成两种或以上不同分辨率的shadow map进行使用的，如图，红色区域是一个高分辨率的shadow map，蓝色区域是覆盖范围更大但分辨率相同也就是更粗糙的shadow map。

![](Games202工业界的实时渲染/Untitled%2015.png)

从而根据物体在场景的位置来选用shadow map，远点的就用粗糙的，近处的就用精细的。但是从图中可以看到会有重叠区域内，因为在突然切换层级时会有一个突变的artifact，为了有一个平滑的过渡，在这个区域内，根据离摄像机的距离把两种shadow map进行blend。

### Cascaded LPV

![](Games202工业界的实时渲染/Untitled%2016.png)

同样的，先在细小的格子上，之后随着距离的增加到大一点的格子，再远就更大，从而省去了很多的计算量。

### Geometric LOD

如果有一个精细的有许多三角形的模型（高模），我们可以将其减化，减少其三角形使其变成一个低模，我们可以生成一系列的低模。

- 可以根据物体离camera的距离，来选择放什么层级的模型进去（高模还是低模）；
- 并且一个物体的不同部分也可以做不同的LOD，只需要提供一个标准，比如说这个标准是保证任何时候三角形都不会超过一个像素的大小。

这样可以保证一个模型可以根据camera在哪，从而动态的告诉渲染管线需要渲染什么层级的模型或者部分。

**问题**：如果camera推进，在推进过程中由于是突变的，会出现突然变化几何的现象，也就是popping artifacts，但TAA可以很好的解决这个现象。

而在UE5中的Nanite技术，本质就是一个复杂的LOD，它所作的就是将LOD存在的问题（不同LOD如何正确连接而不出现缝隙、高模与低模如何进行调度、物体的几何表示用三角形网格还是几何纹理、裁剪与剔除）去寻找已有的论文进行复现与对比，参考：[UE5开发路线图及技术解析-Nanite、Lumen及其它 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/445634734)

# 工业界的全局光照 Global Illumination Solutions

没有任何的GI解决方法可以将所有的情况都给解决掉，除了RTRT（实时光线追踪），因为它理论上是一个正确的path tracing，肯定可以解决各种情况。但是RTRT在现如今只适用于部分，如果整体都用RTRT那么帧率自然而然会下降。

因此工业界经常使用hybrid solutions，也就是将许多中方法结合在一起。

下面是一个可能的GI思路：

- 先做一遍SSR，得到一个近似的GI，对于SSR无法得到的结果，专用其它的ray tracing方法去弥补（用硬件或者软光追）
- 软光追（Software ray tracing）
    - **近处：在任意一个shading point，在其周围的一定范围内的物体用一个较高分辨率或较高质量的SDF，SDF可以让我们在shader中快速地tracing**
    - **远处：用一个稍低质量的SDF将整个场景覆盖，至此不论近处还是远处我们都可以通过SDF得到最终光照打到的结果。**
    - **如果场景中有强烈的方向光源或者点光源（如手电筒），则通过RSM来解决。**
    - 如果场景中有diffuse的物体，则通过DDGI（Dynamic Diffuse GI）来解决，这是一种基于探针的全局光照方法，用探针来存储3D网格中的irradiance，再照亮其它物体。
- 硬件光追（Hardware ray tracing）
    - **使用简化模型。由于我们要计算的是间接光照，因此没有必要去使用原始geometry，可以用一些简化了的模型去代替原始模型从而加快trace的速度，这样RTRT就十分快了。**
    - Probes (RTXGI)，用RTRT的思路结合探针的思路，叫做RTXGI。

上述中加粗的四条结合在一起就是UE5的lumen思路。

# 笔记参考

[Lecture 14 A Glimpse of Industrial Solution_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1YK4y1T7yY/?p=14&vd_source=fe75c02381cf9faf4ece63051ea5c7de)

[GAMES202 Lecture14:Glimpse of Industry Solution - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/397575842)

[【《Real-Time Rendering 3rd》 提炼总结】(七) 第七章续 · 延迟渲染(Deferred Rendering)的前生今世 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/28489928)

[虚幻5的移动端延迟渲染技术到底有多好用？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/562673914)

[剖析虚幻渲染体系（04）- 延迟渲染管线（1/3） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/547943994)

[UE5开发路线图及技术解析-Nanite、Lumen及其它 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/445634734)