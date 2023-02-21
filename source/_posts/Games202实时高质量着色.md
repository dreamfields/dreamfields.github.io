---
title: Games202-06实时高质量着色
tags: 
- CG
categories: 
- - CG 
  - GAMES202
- - Theory
date: 2023-01-04 16:22:25
---
# 实时渲染中的PBR

## 介绍

**PBR（physically-based rendering，基于物理的渲染）** 是指在渲染过程中，材质、光源、相机、光线材质等一切事物都应该基于物理。

因此PBR在概念上来说不只限制在材质上，但是在实时渲染中PBR指的通常是基于物理的材质。

实时渲染中的 PBR 材质丰富程度要远远落后于离线渲染：

- 种类：离线渲染中artist可以找出几百种材质，而在RTR中则有十几种，因为要考虑到速度和研究透彻的程度
- 质量：为了让材质快速被渲染，我们需要牺牲较大的质量效果。比如：毛发的渲染，由于它的超级复杂性，毛发根数众多，如果模拟每根毛发的light transport那将是一个巨大的计算量，会拖慢渲染速度。**实时渲染中速度是首要，在保证速度的前提下，尽可能的提升质量**。
- 物理正确："PB”在实时渲染中由于做了大量的简化来保证速度，因此严格来说**基本都不是基于物理**(Physically-Based)的

## PBR中的材质分类

### 1.基于表面定义的材质

大部分只是微表面模型（Microfacet models），有时候并不是完全基于物理的；还有一种是迪士尼原则性BRDF（Disney principled BRDFs），对艺术家友好，但仍然不是PBR。

### 2.基于**体积上定义的材质**

由于光线会进入到云，烟，雾，皮肤，头发等体积里，在RTR中基于体积上要比基于表面的渲染困难许多，我们大部分考虑的还是光线在这些体积中作用一次(single)和多次(multiple)的分离考虑方法。

### 小结

1. 对于PBR材质来说,在RTR中并没有什么新理论，我们所用的都是离线渲染中仍在使用的，但是放在RTR中就会变得开销巨大。
2. 因此在RTR中我们更多的是考虑用来解决问题的Hacks方法，因为RTR中速度是首要的。

# 基于物理的材质

## 微表面模型 ****Microfacet BRDF****

### 介绍

**微表面模型（microfacet model）** 是一种基于物理的局部光照模型，它假设物体的表面是凹凸不平的，宏观的表面由许多微小的平面（即微表面）构成，光线在每个微平面上发生理想镜面反射或折射。在games101中的材质一节已经讲述过，参见：[Lecture 17 材质（微平面理论，Cook-Torrance BRDF） - Dream Fields](https://dreamfields.github.io/2022/02/08/Lecture-17-%E6%9D%90%E8%B4%A8/#%E5%BE%AE%E5%B9%B3%E9%9D%A2%E7%90%86%E8%AE%BA)

由微表面模型定义的Cook-Torrance**镜面反射**BRDF 如下：

$$
f_{\mathrm{r}}\left(\omega_{i}, \omega_{o}\right)=\frac{F\left(\omega_{i}, h\right) G\left(\omega_{i}, \omega_{o}, h\right) D(h)}{4\left|\omega_{i} \cdot n\right|\left|\omega_{o} \cdot n\right|}
$$

Cook-Torrance BRDF的镜面反射部分包含三个函数，此外分母部分还有一个标准化因子 。字母D，F与G分别代表着一种类型的函数，各个函数分别用来近似的计算出表面反射特性的一个特定部分。三个函数分别为法线分布函数(Normal Distribution Function)，菲涅尔方程(Fresnel Rquation)和几何函数(Geometry Function)：

- F 菲涅尔方程：菲涅尔方程描述的是在不同的表面角下表面所反射的光线所占的比率。以上的每一种函数都是用来估算相应的物理参数的，而且你会发现用来实现相应物理机制的每种函数都有不止一种形式。
- G 几何函数：描述了微平面自成阴影的属性。当一个平面相对比较粗糙的时候，平面表面上的微平面有可能挡住其他的微平面从而减少表面所反射的光线（grassing。
- D 法线分布函数：估算在受到表面粗糙度的影响下，取向方向与中间向量(half vector)一致的微平面的数量。这是用来估算微平面的主要函数。

下面详细介绍各个项的意义。

### ****菲涅耳项 Fresnel Term****

**菲涅耳项**描述了景物表面反射光线的比例依赖于光线的入射角和偏振这一现象。参见：[Lecture 17 材质（微平面理论，Cook-Torrance BRDF） - Dream Fields](https://dreamfields.github.io/2022/02/08/Lecture-17-%E6%9D%90%E8%B4%A8/#%E8%8F%B2%E6%B6%85%E8%80%B3%E5%8F%8D%E5%B0%84fresnel-reflection)

总结来说，可以对菲涅尔项使用 **Schlick's approximation**：

$$
\begin{array}{c}R_{\mathrm{Schlick}}(\theta)=R_{0}+\left(1-R_{0}\right)(1-\cos \theta)^{5} \\R_{0}=\left(\frac{\eta_{i}-\eta_{t}}{\eta_{i}+\eta_{t}}\right)^{2}\end{array}
$$

$R_0$相当于光线垂直入射表面时的反射系数，它的值取决于物体。

对于该近似的改进参见：[《GAMES202：高质量实时渲染》4 实时高质量着色 2.1.1 Fresnel Term](https://zhuanlan.zhihu.com/p/563684531)

### ****微表面的法线分布 Normal Distribution Function****

![](Games202实时高质量着色/Untitled.png)

**微表面法线分布函数（normal distribution function）** 决定了有多大比例的微表面法线朝向 $h$，它是定义在半球面上的三维随机变量服从分布的概率密度函数。

- 分布的数学期望是 $h$
- 分布的方差越小、越集中，则描述的材质越接近于有光泽的材质（glossy），
- 分布的方差越大、越分散，则描述的材质越接近于漫反射材质（diffuse）。

我们有有很多不同的模型来描述法线分布：常用的Beckmann，GGX等等模型；Yan 等人提出的一系列法线分布函数（NDF）。下面介绍其中几种：

**Beckmann法线分布函数**

由定义在坡度空间（slope space）上的**正态分布导出：**

$$
D_{\text {Beckmann }}(h)=\frac{e^{-\frac{\tan ^{2} \theta_{h}}{\alpha^{2}}}}{\pi \alpha^{2} \cos ^{4} \theta_{h}} 
$$

- $\alpha$是表面的粗糙程度，不同粗糙程度的意思是NDF中lobe是集中在一个点上，还是分布的比较开。类比于高斯函数中用 $\sigma$来控制胖瘦，也就是标准差。
- $\theta_h$ 是微表面法线和宏观表面法线之间的夹角

由公式可以看出：

1. 该函数的定义只与$\theta$有关，与$\phi$无关，因此表述的是各向同性的结果，也就是沿着中心旋转是相同的结果。
    
    ![](Games202实时高质量着色/Untitled%201.png)
    
2. 公式中分子的指数中用的是$tan \theta_h$，因为是定义在Slope space 坡度空间上的，如下图。竖直向上的是宏观法线，微表面的法线延长线与宏观法线的切平面相交。这也保证了在$\theta_h$的定义域内这个值永远有意义：由于高斯函数的定义域是非常大的，但在过了3 $\sigma$之后会缩减到非常小但不是0，为了满足这一性质定义在坡度空间，从而保证在slope space中无限大的函数无论如何也不会出现面朝下的微表面。

![](Games202实时高质量着色/Untitled%202.png)

**GGX法线分布函数**

又称 Trowbridge-Reitz 法线分布函数，它的公式如下：
$$
D_{\mathrm{GGX}}(h)=\frac{\alpha^{2}}{\pi \cos ^{4} \theta_{h}\left(\alpha^{2}+\tan ^{2} \theta_{h}\right)^{2}}
$$
化简后的公式如下：
$$
D_{\mathrm{GGX}}(h)=\frac{\alpha^{2}}{\pi ((n \cdot h)^2(\alpha^{2}-1)+1)^2}
$$

其中：$ \alpha = roughness^2 $，$h$是半程向量

和 Beckmann 法线分布函数相比，GGX 法线分布函数更“长尾”（long tail），也就是说，当随机变量的取值偏离其数学期望时，相应的概率下降得更慢一些，如下图：

![](Games202实时高质量着色/Untitled%203.png)

这会带来两个好处:

1. Beckmann的高光会逐渐消失，而GGX的高光会减少但不会消失，这就意味着高光的周围我们看到一种光晕的现象。
2. GGX除了高光部分，其余部分会像Diffuse的感觉。

![](Games202实时高质量着色/Untitled%204.png)

**GTR法线分布函数（GGX的扩展）**

Brent Burley 在总结 Disney Principled BRDF 时，提出了更“长尾”的 GTR 法线分布函数（Generalized-Trowbridge-Reitz），它的公式如下：

$$
D_{\mathrm{GTR}}(h)=\frac{c}{\left(\alpha^{2} \cos ^{2} \theta_{h}+\sin ^{2} \theta_{h}\right)^{\gamma}}
$$

- 其中，$c$ 是数据归一化常数；
- 增加了参数$\gamma$，根据其值的不同可以调节拖尾长度。当 $γ=2$ 时，$c=α^2/π$，此时 GTR 退化为 GGX；超过10时会接近Beckmann。

![](Games202实时高质量着色/Untitled%205.png)

在实时渲染中，除非特殊场景（例如雪地），出于效率的考虑，一般还是使用相对简单的法线分布函数。

### 阴影遮蔽项 ****Shadowing-Masking Term****

**阴影遮蔽项（shadowing-masking term）** 又称几何项（geometry term），是光能由于微表面之间相互遮挡（自遮挡）而衰减的系数。

- 当光线入射方向或观察方向几乎垂直于景物表面时，微表面之间差不多没有相互遮挡，几何项接近于1；
- 当光线入射方向或观察方向几乎平行于景物表面、接近于掠射（grazing angle）时，微表面之间相互遮挡的程度很大，几何项接近于0。

![](Games202实时高质量着色/Untitled%206.png)

如上图，有两种情况：

- 左边这中从light出发，发生的微表面遮挡现象叫做Shadowing
- 右边这种从eye出发，发生的微表面遮挡现象被称为Masking

效果：由于微表面的自遮挡，实际计算出的结果会比理想结果亮，所以加上G项使得结果变暗接近理想结果。

引入G项的理论分析：

$$
f_{\mathrm{r}}\left(\omega_{i}, \omega_{o}\right)=\frac{F\left(\omega_{i}, h\right) G\left(\omega_{i}, \omega_{o}, h\right) D(h)}{4\left|\omega_{i} \cdot n\right|\left|\omega_{o} \cdot n\right|}
$$

对于整个BRDF，如果从grazing angle看，不考虑G项时，分子为正常函数，没有很大的值。对于分母，当在grazing angle时入射方向、出射方向与法线角度接近90°，因此点乘结果接近0，分子除以一个接近于0的值会导致结果变的巨大，就会导致我们看到的这张图整个外圈是白的。

![](Games202实时高质量着色/Untitled%207.png)

**G项和法线分布函数有关，可由其进行预测**，在 Smith shadowing-masking term 的假设下，双向的阴影遮蔽项被拆解成了Shadowing阴影项和Masking遮蔽项：

$$
G_{Smith}\left(\omega_{i}, \omega_{o}, n\right)=G_{Schlick}\left(\omega_{i}, n\right) G_{Schlick}\left(\omega_{o}, n\right)
$$

$$
G_{Schlick}(v,n)=\frac{n\cdot v}{n\cdot v(1-k)+k}
$$

$$
k=\frac{(roughness+1)^2}{8}
$$

下图是绿线是GGX，红线是Beckmann所预测的G项结果（右图），在grazing angle下G项接近0，除以BRDF中接近0的分母会得到一个正常值，解决了外圈发白的问题。

![](Games202实时高质量着色/Untitled%208.png)

## Kulla-Conty BRDF

### 问题引入

微表面模型的 BRDF(Microfacet BRDF) 存在一个根本问题，如下图的白炉测试：

![](Games202实时高质量着色/Untitled%209.png)

上图第一行：随着粗糙程度变大，我们渲染得到的结果却越暗

上图第二行：在uniform的环境光照（各处光照一样）下如果能量没有损失，则渲染出的物理应该和背景同色，可以看到，物体越粗糙，会越暗。

这个根本问题就是：**忽略了微平面间的多次弹射**，这就导致了**材质的能量损失**，并且当材质的粗糙度越高时，能量的损失会越严重。

解决的核心思路：

- 当反射光不被遮挡的时候，这些光就会被看到；
- 当反射光被微表面遮挡的时候，认为**这些被挡住的光将进行后续的弹射**，直到能被看到

### **The Kulla-Conty Approximation**

Heitz 等人在论文中提出了精确的补偿方法，但是效率对于实时渲染而言太低了。为了适用于实时渲染，Kulla 和 Conty 提出了一种较为简单且高效的方法，也就是**The Kulla-Conty Approximation**——通过引入一个**经验性的**微表面BRDF 的补偿项（Energy Compensation Term），来补偿光线的多次弹射，使得材质的渲染结果可以近似保持能量守恒。

需要考虑以下两点：

- 反射时有多少能量丢失了？
- 最后反射出的能量有多少?

### 补偿项推导与计算

1. **首先计算最后反射出多少能量**

$$
\begin{array}{l}E\left(\mu_{o}\right)=\int_{0}^{2 \pi} \int_{0}^{\frac{\pi}{2}} f\left(\mu_{o}, \mu_{i}, \varphi\right) \cos \theta \sin \theta \mathrm{d} \theta \mathrm{d} \varphi \\\stackrel{\cos \theta \mathrm{d} \theta=\mathrm{d} \sin \theta}{\Longrightarrow} \int_{0}^{2 \pi} \int_{0}^{1} f\left(\mu_{o}, \mu_{i}, \varphi\right) \sin \theta \mathrm{d} \sin \theta \mathrm{d} \varphi \\\stackrel{\mu=\sin \theta}{\Longrightarrow} \int_{0}^{2 \pi} \int_{0}^{1} f\left(\mu_{o}, \mu_{i}, \varphi\right) \mu_{i} \mathrm{d} \mu_{i} \mathrm{d} \varphi \\\end{array}
$$

上式的计算要注意：

- 只计算上半球所以$\theta$的积分域为$(0,\pi/2)$；
- 单位立体角大小是$\sin \theta \ \theta \mathrm{d} \varphi$ ，与$BRDF \cdot cos\theta$放在一起进行积分
    
    ![](Games202实时高质量着色/Untitled%201.png)
    
- 我们认为任何方向入射的Radiance是1，也就是rendering equation中的Lighting项是1（因为是1所以式子中没有出现）
- 假设BRDF是各向同性，与入射光、反射光方向无关
- 最终积分的意义是，uniform的lighting=1的情况下，在经历了1 次bounce之后射出的总能量
1. **计算补偿项**

由于最后反射出的总能量$E\left(\mu_{o}\right)$在0-1之间，那么在方向$\mu_o$上损失的能量为$1-E\left(\mu_{o}\right)$，于是在所有方向上反射出来的平均能量$E_{avg}$为：

$$
E_{avg }=\frac{\int_{0}^{1} E(\mu) \mu \mathrm{d} \mu}{\int_{0}^{1} \mu \mathrm{d} \mu}=2 \int_{0}^{1} E(\mu) \mu \mathrm{d} \mu
$$

由于要补偿的BRDF也具有可逆性，还要考虑入射时所丢失的能量，于是乘上另外一项$1-E\left(\mu_{i}\right)$，因此Kulla 和 Conty提出的最终的BRDF补偿项$f_{ms}$为：

$$
f_{ms}\left(\mu_{o}, \mu_{i}\right)=\frac{\left(1-E\left(\mu_{o}\right)\right)\left(1-E\left(\mu_{i}\right)\right)}{\pi\left(1-E_{avg}\right)}
$$

可以验证其积分的结果是正确的，就是损失的能量：

![](Games202实时高质量着色/Untitled%2010.png)

1. **预计算$E_{avg}$——打表**

由于$E(\mu_o)$已经是一个二重积分了，计算仍然比较复杂，而对于一个很复杂的、不一定有解析解的积分可以通过**预计算或打表格**的方式来解决。

其中 $E(\mu)$ 随粗糙度和角度两个变量变化，而 $E_{avg}$ 仅随粗糙度变化，通过预计算可以得到类似于下图的表：

![](Games202实时高质量着色/Untitled%2011.png)

到此，就可以将计算得到的损失的能量加上去，效果如下：

![](Games202实时高质量着色/Untitled%2012.png)

1. **颜色材质的计算**

**思路**：对于有颜色的材质，会有自然存在的能量损失，这样就需要在之前计算得到的 $f_{ms}(\mu_{o},\mu_{i})$ 上乘上一个附加的 BRDF 项 $f_{add}$，也就是先假设没有颜色来计算补偿项，然后考虑物体对颜色的吸收（吸收了其它波长的光，所以才会反射出某种颜色）产生的损失。

**定义引入**：平均的菲涅尔项$F_{avg}$，该项描述了——假设在材质中的微平面之间发生了多次反射，不管入射角多大，平均每次发生反射会有多少比例的能量反射出去。也就是说，**在计算没有颜色的补偿项的时候，默认菲涅尔项是1**。该项的计算公式为：

$$
F_{avg}=\frac{\int_{0}^{1} F(\mu) \mu \mathrm{d} \mu}{\int_{0}^{1} \mu \mathrm{d} \mu}=2 \int_{0}^{1} F(\mu) \mu \mathrm{d} \mu
$$

于是，光线入射到表面后，发生如下过程（$E_{avg}$表示从表面反射出来被人眼看到的平均能量）：

- **初次反射**，能被直接看到的能量：

$$
E_0 = F_{avg}E_{avg}
$$

（1）首先假设入射能量为1，有$F_{avg}$的比例能够反射；

（2）发生反射的那部分有$E_{avg}$的能量离开了表面被人眼看到，因此相乘；

---

- **在微平面之间反射（可以扩充为散射） 1 次后**，看到的能量：

$$
E_1 = F_{avg}(1-E_{avg}) \cdot F_{avg}E_{avg}
$$

（1）初次反射没有被直接看到的能量为$(1-E_{avg})$，意味着反射到了另一个微表面，那么反射到另一个微表面的能量是$F_{avg}(1-E_{avg})$；

（2）在另一个微表面上，入射的能量就是$F_{avg}(1-E_{avg})$，有$F_{avg}$的比例能够反射；

（3）发生反射的那部分有$E_{avg}$的能量离开了表面被人眼看到，因此相乘；

---

- 在微平面之间散射 2 次后，看到的能量比例：

$$
F_{avg}(1-E_{avg}) \cdot F_{avg}(1-E_{avg}) \cdot F_{avg}E_{avg}
$$

---

- ……

---

- 在微表面之间散射k次后，看到的能量比例：

$$
F_{avg}^k(1-E_{avg})^k  \cdot F_{avg}E_{avg}
$$

将这些项相加，得到最终的$f_{add}$：

$$
f_{add}=\frac{\sum_{i=1}^{\infty} F_{avg }^{k}\left(1-E_{avg }\right)^{k} \cdot F_{avg } E_{avg }}{1-E_{avg }}=\frac{F_{avg } E_{avg }}{1-F_{avg }\left(1-E_{avg }\right)}
$$

最后对于 Kulla-Conty 材质，可以得到的 BRDF 项 $f_r$ 为：

$$
f_r = f_{micro}+f_{add}*f_{ms}
$$

其中$f_{micro}$是原本的微表面模型定义的BRDF，后面是补偿项；$E_{avg}$ 、补偿系数 $f_{ms}$ 和菲涅尔项均值 $F_{avg}$ 都可以在开始绘制前预计算，在解绘制方程时直接调用相应的数值。

到此，效果如下：

![](Games202实时高质量着色/Untitled%2013.png)

Kulla-Conty 方法在物理上是正确的，但是有时为了方便，会直接使用漫反射材质的BRDF补偿项，则能量不守恒，在物理上是错误的。不过，有一些研究在考虑构造出能量守恒的漫反射波瓣的方法。

## ****Disney’s Principled BRDF****

微表面模型的问题：

1. 微表面模型能够描述材质有限，像是刷上一层清漆的木桌表面这样的材质，单独地使用微表面模型就难以建模。因为微表面模型最多解释一层材质，而无法解释多层材质。
2. 基于物理的模型直接使用折射率这样的物理量作为参数，而这些参数和材质最终呈现的效果没有直观的联系，对艺术家并不友好。

一些材质在建模时为了让艺术家用起来顺手，物理上的正确性就没有那么的必要了。Disney Principled BRDF 便是在满足能量守恒，物理光学，以及艺术家用起来方便等条件下妥协的产物，但是在RTR中我们认为Disney's principle BRDF也算是PBR材质。

它有几个重要的设计原则：

- 使用直观的参数，而不是物理参数来控制模型呈现的效果，比如使用平缓、饱和度等；
- 模型使用的参数尽可能的少；
- 参数的合理范围应该是从 0 到 1；
- 在必要时，即使参数的取值超出了 [0,1] 范围，也能得到有意义的结果；
- 所有的参数组合应该尽可能的稳健和合理；

下图的表格独立显示了各个参数在不同数值下的效果，在实际操作时这些参数是可以组合、叠加起来：

![](Games202实时高质量着色/Untitled%2014.png)

**Disney's principle BRDF的优点:**

1. 容易理解和使用各参数(属性)
2. 参数的混合组合使得可以在一个模型上显示出很多不同的材质
3. 开源

**Disney's principle BRDF的缺点:**

1. 并不是完全基于物理的
2. 巨大的参数空间使得拥有强大的表示能力，但是会造成冗余现象

# 基于物理的着色

## ****LTC（Linearly Transformed Cosines）****

根据多边形光源着色物体表面，需要在多边形覆盖的区域中对定义在球面上的 BRDF 计算定积分，但是直接计算定积分相当困难。

**LTC（Linearly Transformed Cosines）** 即在多边形光源的照明下的着色微表面模型，该算法通过转换多边形光源及微表面模型的散射波瓣（Lobe），在**不考虑可见性**的情况下快速地求解出渲染方程的解析解。

![](Games202实时高质量着色/Untitled%2015.png)

**特性**：

1. 主要是针对GGX模型，对于其他模型原理也同样适用
2. 做的是**不考虑shadow**的shading
3. **光源是多边形光源**，且发出的radiance时uniform的
4. **LTC是避免在多边形光源上采样的方法，Split Sum是避免在环境光上采样的方法**

**关于上面提到的Lobe**：

Lobe就是在固定一边方向下的BRDF的函数图象，由于BRDF是一个4维的函数，在固定一边之后就变为了2维的函数。大多数高光BRDF，都是在半角向量接近宏表面法线时反射亮度最大，向上或向下时其值都会减小,使得其函数图象像叶片，故称为lobe。

**LTC方法的思路**：

在给定了观察方向 $ω_o$ 后，对于任何 BRDF 的散射波瓣 $F(ωi)$，都可以通过一个线性变换 $M^{-1}$得到余弦函数描述的波瓣 $cos⁡ω′_i$ ，同时，该变换也将光的入射方向、多边形光源（也就是渲染方程的积分域）进行变换。

$cos(\theta_s)$是一个球面分布函数（**余弦分布函数**），$f(p,wi,wo)$也是一个球面分布函数，因此其实我们是让一个球面分布函数通过线性变换变成了另一个球面分布函数。

![](Games202实时高质量着色/Untitled%2016.png)

因此，该方法通过线性变换$M^{-1}$，使得一个多边形光源（积分域为$P$）照亮某个物体产生的lobe（2D BRDF Lobe），等价于——用一个新的多变形光源（$P'$）去照亮一个余弦函数定义的lobe（Cosine Lobe）来替换，且这样一种方法是能够得到解析解的（**也就是不需要采样，将数值代入就可以得到结果**）。

于是就可以**将shading point点任意BRDF的Lobe在任意多边形光源下积分求shading的问题转变为在一个固定cos函数下对任意的多边形光源积分求shading**。

**LTC方法的步骤**：

![](Games202实时高质量着色/Untitled%2017.png)

- 首先**预计算**一系列不同观察方向和粗糙度下 Microfacet BRDF 转换到余弦波瓣的线性变换  $M^{-1}$
- 然后在着色时查表，根据线性变换  $M^{-1}$转换光线入射方向立体角积分微元 $\mathrm{d}w_i$和多边形光源积分域 $P$
- 最后对转换为余弦波瓣的 BRDF 计算定积分。

定积分解析解的推导：[Geometric Derivation of the Irradiance of Polygonal Lights (archives-ouvertes.fr)](https://hal.archives-ouvertes.fr/hal-01458129/document)

**效果**：

![](Games202实时高质量着色/Untitled%2018.png)

![](Games202实时高质量着色/Untitled%2019.png)

# ****非真实感渲染 NPR****

![](Games202实时高质量着色/Untitled%2020.png)

对比于真实感绘制模拟景物在真实环境中的光照效果，生成犹如照片的图像，**非真实感渲染（Non-Photorealistic Rendering，NPR）** 模拟艺术式的绘制风格，生成风格化的图像。

在实时渲染中，NPR 的目标是快速而可靠地生成风格化的结果，通常使用一些轻量级的解决方法，在着色器中进行一些简单而巧妙的处理。

**一般的研究思路**：

从真实感绘制出发，进行合理地抽象，强调重要的部分，得到风格化的结果。**卡通风格 NPR** 通常描边以强调物体的轮廓（outline），使用分界明显的色块，而不是平滑过渡的颜色。**素描风格 NPR** 则通过不同疏密的线条纹理来生成结果。

## 轮廓渲染 O****utline rendering****

通常所讲的“描边”（contours）是轮廓渲染的一个子集。

![](Games202实时高质量着色/Untitled%2021.png)

如图，将一个盒子细分为各种类型的"边缘"：

- 【B】Boundary/border edge 物体外边界上
- 【C】Crease 折痕，通常是在两个表面之间的
- 【M】Material 材质边界
- 【S】Sihouette edge 必须是在物体外边界上且由多个面共享的

下面介绍几种描边方法。

### 基于法线的描边 Shading normal contour edges

为了强调物体的轮廓，可以在着色时考虑描边的效果。一个简单的处理方法是：

- 设定阈值
- 判断：法线方向与观察方向近乎于垂直的表面更暗一些，来得到Sihouette edge

不过，这样得到的描边效果可能粗细不一，原因在于通过判断角度来进行描边操作，无法固定边的粗细，因此法线变化平缓的地方描边结果就比较粗，法线变化尖锐的地方就会比较细，如下图：

![](Games202实时高质量着色/Untitled%2022.png)

### 基于几何的描边

当我们去渲染一个物体/模型时候，如果我们把模型加大一圈，然后大模型放在原模型背后并且把新模型完全渲染成黑，就得到了描边的效果。

在实际操作中，有更有效的方法：将背面的每个顶点沿着顶点的法线方向向外偏移，从而达到背面外拓，即让模型的背面比正面更大一些，并在着色时指定背面的颜色是黑色的。于是，绘制出的模型正面就会带有一圈黑边，对应于延伸出的模型背面。

![](Games202实时高质量着色/Untitled%2023.png)

特性：

- 简单稳定，不需要获取三角形的邻接信息，不过只能显示Contour edge
- 需要将整个背部的面都渲染，所以这种方式可能造成一些资源浪费
- 对于半透明物体的渲染描边，也会产生影响

### **基于图像的后期处理的描边**

基本思路：先生成未描边的正常渲染结果，再通过一些图像处理的方法从而得到这些边并且标上颜色。

以Sobel detector为例，他的做法是做了图像的filter(卷积)，设计filter kernel得到我们想要的边：

![](Games202实时高质量着色/Untitled%2024.png)

## 色块 Color blocks

分界明显的色块效果，可以通过：

- 首先得到正常的Shading model算出来的结果
- 阈值化操作

阈值化操作可以设定多个值来获得更多的色块：

![](Games202实时高质量着色/Untitled%2025.png)

也可以根据自己需要去将不同风格的结果组合在一起，如diffuse和specular都进行阈值化处理，或者只处理spcular不处理diffuse等组合：

![](Games202实时高质量着色/Untitled%2026.png)

## 素描风格 Strokes Surface Stylization

有时候我们并不想得到色块的感觉，而是想得到素描一样的效果：

![](Games202实时高质量着色/Untitled%2027.png)

一个直观的思路：在素描上，往往越暗的地方需要更多的涂抹，我们把它理解成小格子，越暗的地方格子越多也就是密度越大，因此我们把格子密度与明暗度联系在一起。

在 2001年，Praun 等人发表了论文《[Real-time hatching](https://hhoppe.com/hatching.pdf)》提出了一种生成素描风格图像的方法，用不同疏密的线条来表示模型表面的起伏。提出了 TAMs（tonal art maps）方法进行着色，实现类似手绘素描风格渲染。

TAMs 由一系列密度不同的线条纹理组成，而各个不同密度的线条纹理都有各自的 MIPMAP：

![](Games202实时高质量着色/Untitled%2028.png)

算法分为两步：

1. 关于不同明暗设计不同的纹理
2. 对于不同明暗的纹理设计其密度不变的mipmap，利用密度不变来保证距离远近不改变纹理的明暗。原因在于：如果物体开始远离camera，由于纹理是贴在物体上的，由距离越远导致物体变小，如果纹理映射不改变，纹理密度就会变大。因此，在着色时，就可以根据着色点的明暗及位置选取合适的纹理。

![](Games202实时高质量着色/Untitled%2029.png)

# 参考

[Lecture9 Real-time GLobal Illumination (screen space cont.)_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1YK4y1T7yY?p=9)

[Lecture10 Real-Time Physically-based Materials (surface models)_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1YK4y1T7yY?p=10)

[《GAMES202：高质量实时渲染》4 实时高质量着色：Microfacet Model、LTC（Linearly Transformed Cosines）、非真实感渲染（NPR） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/563684531)

[GAMES202 高质量实时渲染笔记Lecture10：Real-Time Physically-Based Materials (Surface models) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/384559264)

[GAMES202 高质量实时渲染笔记Lecture11：Real-Time Physically-Based Materials (Surface models cont.) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/385438367)

# 论文

【离线渲染下补偿能量损失】[Multiple-Scattering Microfacet BSDFs with the Smith Model](https://link.zhihu.com/?target=https%3A//eheitzresearch.wordpress.com/240-2/)

【Kulla Conty】[https://blog.selfshadow.com/publications/s2017-shading-course/imageworks/s2017_pbs_imageworks_slides_v2.pdf](https://blog.selfshadow.com/publications/s2017-shading-course/imageworks/s2017_pbs_imageworks_slides_v2.pdf)

【定积分解析解的推导】[Geometric Derivation of the Irradiance of Polygonal Lights (archives-ouvertes.fr)](https://hal.archives-ouvertes.fr/hal-01458129/document)

【Praun TAMs】[https://hhoppe.com/hatching.pdf](https://hhoppe.com/hatching.pdf)