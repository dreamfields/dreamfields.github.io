---
title: 论文综述-Bim减面算法调研
tags:
  - Algorithm
  - 论文综述
categories:
  - - CG
    - 论文综述 
  - - Algorithm
  - - Paper reviews
date: 2022-06-25 10:54:15
---

# 相关概念

BIM：建筑信息模型（Building Information Modeling）

Lod：多细节层次（Levels of Detail），指根据物体模型的节点在显示环境中所处的位置和重要度，决定物体渲染的资源分配，降低非重要物体的面数和细节度，从而获得高效率的渲染运算。

Revit：该系列软件是为[建筑信息模型](https://baike.baidu.com/item/%E5%BB%BA%E7%AD%91%E4%BF%A1%E6%81%AF%E6%A8%A1%E5%9E%8B/5034795)(BIM)构建的，可帮助建筑设计师设计、建造和维护质量更好、能效更高的建筑，常被用来做二次开发达到想要的功能。

# 论文综述

---

> **C Q L A , B X S , C B Y , et al. Geometric structure simplification of 3D building models[J]. ISPRS Journal of Photogrammetry and Remote Sensing, 2013, 84(10):100-113.**

[Geometric structure simplification of 3D building models.pdf](Bim减面算法调研/Geometric_structure_simplification_of_3D_building_models.pdf)

本文提出了一种结构简化的方法。将建筑模型的**几何结构分为三类：嵌入结构、组成结构和连接结构，可以通过凸/凹分析分别提取**。本文还**提出了一些具体的几何结构简化规则**，并建议对建筑模型进行逐步的简化。实验证明了该方法的稳健性和高效性，并说明了建筑模型细节程度的应用。第 3 节中，我们介绍了**表面补丁外置（Surface patch extraction）算法**，作为将三⾓形模型转换成多边形模型的预处理步骤。在第 4 节中，我们讨论了建筑模型的⼏何结构的分类，并描述了⼀种稳健的⼏何结构提取⽅法的去尾。第 5 节阐述了**结构简化规则并提出了建筑模型的渐进式简化⽅法**。第 6 节介绍并讨论了实验结果。

![Untitled](Bim减面算法调研/Untitled.png)

**本研究所参考的相关工作总结如下：**

鉴于建筑物在城市环境中的重要作⽤，地理信息科学领域的科学家对建筑物模型的简化最感兴趣。然⽽，这与 CAD 和计算机图形领域的⼀般模型的简化不同。为了实现⾼效的虚拟环境可视化，研究⼈员更多地受到制图学思想的影响，**正则化和聚合的算法被广泛用于远距离建筑模型的简化过程，这被称为三维模型泛化。**参照⼆维地图的研究，⼀些应⽤⽅法将建筑模型的简化过程分为两个阶段：墙体（或脚印）的简化和屋顶的简化。在⼀项早期的研究中，Kada（2002）提出了⼀种基于最⼩⼆乘法调整理论的简化⽅法，并结合⼀套精⼼设计的表⾯分类和简化操作。该⽅法的主要缺点是只有建筑模型表⾯的⼀些简单挤压特征可以被简化。作为对以前⽅法的改进，他建议使⽤类似于半空间建模的过程对物体进⾏重塑（Kada, 2006），并提出了⼀个两阶段的算法（Kada, 2007），该算法结合了原始实例化和单元分解。同时，受图像分析中标度空间理论的启发，Mayer（2005）和 Forberg（2007）通过结合数学形态学和曲率空间，提出了⼀种新的建⽴模型的概括⽅法。

与上述⽅法相⽐，也有从**几何简化的角度**开发的三维建筑模型的算法。Rau 等⼈（2006）将建筑模型视为⼀组相连的多⾯体，并提出了⼀种通过改变特征分辨率⾃动⽣成伪连续 LOD 的算法。Du 等⼈（2008）和 Zhu 等⼈（2009）提出了在⼈类视觉系统（HVS）的指导下识别⼆维图像空间中复杂物体的⼏何特征并简化建筑模型。Jiang 等⼈（2011 年）开发了⼀种新的⽅法，通过细胞聚类和重建⽣成三维复杂建筑模型的多分辨率代表。Sun 等⼈（2011 年）提出了⼀种⾃动算法，通过凸聚类⽣成建筑模型的结构保护性抽象。

另⼀⽅⾯，为了保证简化后建筑模型的识别⼀致性，⼈们对**建筑模型中各组成部分的语义或几何关系**给予了⼤量关注。Fan 等⼈（2009）提出了 CityGML 中三维建筑模型的语义⽣成⽅法。Zhao 等⼈（2012）最近提出了⼀种基于数学形态学的泛化⽅法，该⽅法通过考虑建筑构件之间的语义关系，在多个 LoD 中保持⼏何、拓扑关系和语义的⼀致性⽅⾯有很⼤的能⼒。然⽽，他们的⽅法都对数据集提出了严格的要求。对于⼀般的建筑模型，Thiemann 和 Sester(2004)曾经提出了⼀种表⾯分割⽅法，以检测和删除表⾯的细节特征来简化模型，但他们的⽅法实际上被认为是⼀种理论上的简化⽅案，只适合结构⾮常简单的建筑模型。

---

> **Li M , Nan L . Feature-preserving 3D mesh simplification for urban buildings[J]. ISPRS Journal of Photogrammetry and Remote Sensing, 2021, 2021(173):135-150.**

[Feature-preserving 3D mesh simplification for urban buildings.pdf](Bim减面算法调研/Feature-preserving_3D_mesh_simplification_for_urban_buildings.pdf)

该篇研究的目标是建立一个全自动管道，在保持 BIM 建筑结构的同时缩小城巿建筑网格模型。该方法的目的不是一次处理整个城市场景，而是将场景中的单个建筑物逐一进行处理。

![Untitled](Bim减面算法调研/Untitled%201.png)

该研究所提出的全自动管道由三个主要模块组成：网格过滤、结构提取和网格抽取。简化可以滤除小尺度细节，结果可以形成多分辨率层次结构，以进行高效的几何处理以及细节级别(LOD)生成。
该研究的主要贡献有两个:

一是提出了一种网格滤波方法，可以逐步增强分段平滑度和锐利的轮廓特征。从而可以利用简单的区域增长算法从网格中提取平面形状，网格的结构由拓扑图表示，拓扑图由一组平面区域和经过网格过滤后检测到的轮廓内边缘组装而成。

二是提出了一种基于层次策略的细化网格抽取方法。与传统的网格抽取方法相比，使用了不同的策略为平面和非平面区域中的边缘塌陷算子设计误差度量。在上一步中检测到的平面用作约束，以避免在边缘折叠迭代期间错误累积。因此，简化模型尽可能地保留了原来的平面结构。

达到的效果如下图所示，描述了航空摄影测量模型的简化过程。从(a)到(f)分别为：初始网格，网格过滤结果，区域增长结果，提取的平面区域结构，轮廓特征，以及最终的简化结果。

![Untitled](Bim减面算法调研/Untitled%202.png)

---

**Anumba C . A Semantics-Based Approach for Simplifying IFC Building Models to Facilitate the Use of BIM Models in GIS[J]. Remote Sensing, 2021, 13.**

[A Semantics-Based Approach for Simplifying IFC Building Models to Facilitate the Use of BIM Models in GIS.pdf](Bim减面算法调研/A_Semantics-Based_Approach_for_Simplifying_IFC_Building_Models_to_Facilitate_the_Use_of_BIM_Models_in_GIS.pdf)

本文首先提出：使用实体建筑模型，而不是城市地理标记语言（CityGeography Markup 语言，CityGML），可以促进建筑信息模型（BIM）和地理信息系统（GIS）之间的数据整合。然而，使用实体模型会带来一个问题模型简化的问题。本研究的目的是通过**开发一个从 BIM 生成简化实体建筑模型的框架**来解决这个问题。

在这个框架中，首先定义了一套适合实体建筑模型的细节级别（LoD）——被称为 s-LoD，范围从 s-LoD1 到 s-LoD。在实现 s-LoD 的过程中发现了三个独特的问题，并通过使用基于语义的方法加以解决，包括：识别 s-LoD2 和 s-LoD3 的外部对象，区分各种板块，以及为 s-LoD2 和 s-LoD3 生成有效的外墙。该框架的可行性通过使用 BIM 模型得到了验证，结果表明，使用 BIM 的语义可以使建筑模型的转换和简化更容易，这反过来又使 BIM 信息在地理信息系统中更加实用。

三种建筑的 LoD 结果：

![Untitled](Bim减面算法调研/Untitled%203.png)

其中一个几何简化的成果：

![Untitled](Bim减面算法调研/Untitled%204.png)

---

> **Yannick, Verdie, Florent, et al. LOD Generation for Urban Scenes[J]. ACM Transactions on Graphics (TOG), 2015, 34(3):1-14.**

[LOD Generation for Urban Scenes.pdf](Bim减面算法调研/LOD_Generation_for_Urban_Scenes.pdf)

本文引入了一种新的方法，以细节层次模型（LODs）的形式重建三维城市场景。从原始数据集开始，例如由多视角立体系统产生的表面网格，该算法分三个主要步骤进行：分类、抽象和重建。

分类：从几何属性和一组与马尔科夫随机场相结合的语义规则，将场景分为四个有意义的类别。

![Untitled](Bim减面算法调研/Untitled%205.png)

抽象：检测并规范建筑物上的平面结构，在树木、屋顶和外墙上拟合图标，并为 LOD 的生成执行过滤和简化，生成抽象出来的数据然后被提供给重建。

重建：通过最小切割公式（min-cut formulation）生成顶层完整的建筑。

最终的效果图如下图所示，第一行：在这个简单的住宅场景中，所有的外墙和屋顶都被很好地分类；第二行：展示的是较为密集的城市，每个建筑的房顶都是简单的，但所有的屋顶形成一个复杂的排布。第三行：在这个建筑上，Z 对称性（Z-symmetry）和正交关系（orthogonal relationships）共同来抽象出屋顶的中心部分。第四行：这个建筑包含复杂而薄的屋顶上部结构。尽管输入的 MVS 网格的精度有限，但本研究的算法还是恢复了主要的外墙和屋顶，以及大部分的上部结构。

![Untitled](Bim减面算法调研/Untitled%206.png)

---

### 以下文献略作参考

> **【2008】Du Z , Zhu Q , Zhao J . PERCEPTION-DRIVEN SIMPLIFICATION METHODOLOGY OF 3D COMPLEX BUILDING MODELS[J]. isprs.**

[PERCEPTION-DRIVEN_SIMPLIFICATION_METHODOLOGY_OF_3D_COMPLEX_BUILDING_MODELS.pdf](Bim减面算法调研/PERCEPTION-DRIVEN_SIMPLIFICATION_METHODOLOGY_OF_3D_COMPLEX_BUILDING_MODELS.pdf)

针对传统简化方法的缺点--难以定位需要简化的部件和基元，本文提出了一种新的简化方法，基于渲染的图像分析，定位需要简化的模型部分。其主要思想是将复杂物体的几何特征在三维物体空间的识别和评估转换到二维图像空间，并使用基于**人类视觉系统（HVS）**模型的图像过滤来指导三维空间的简化操作。

> **Chen K , Chen W , Cheng J , et al. Developing Efficient Mechanisms for BIM-to-AR/VR Data Transfer[J]. Journal of Computing in Civil Engineering, 2020, 34(5):04020037.**

[Developing Efficient Mechanisms for BIM-to-AR_VR Data Transfer(ASCE)CP.1943-5487.0000914.pdf](<Bim减面算法调研/Developing_Efficient_Mechanisms_for_BIM-to-AR_VR_Data_Transfer(ASCE)CP.1943-5487.0000914.pdf>)

这篇文章整体上研究了从 BIM 模型到 AR/VR 数据传输的高效机制，以更好地利用 BIM 的信息。在这个过程中，该研究将**几何模型中的建筑构件根据其特征**进行分类，并**采用不同的多边形缩减算法进行简化**，所提出的机制能够有效地将 BIM 的语义信息转移到 AR/VR 中，大大减少了几何模型的三角形数量，同时最大限度地提高整体形状的一致性，并提高相应的 AR/VR 应用的帧率。

本研究指出，由 BIM 软件生成的一些建筑组件具有大量冗余多边形，可以在保持原始形状完全相同的情况下进行合并。—些弯曲的部件可以大大简化，同时最大限度地保持整体形状的一致性。例如,(b)中的墙的前表面可以用两个三角形表示，其中包含许多冗余多边形。(c)中圆柱体侧面的许多三角形可以合并,而圆柱体的形状几乎是恒定的。

![Untitled](Bim减面算法调研/Untitled%207.png)

本文的重点在于，对于几何模型，将建筑构件分为四种类型，据此提出了不同的多边形缩减方法：

（1）简单构件指的是由最简化的平面组成的构件，因此不需要对其进行多边形缩减；

（2）相互连接的墙体有大量不必要的三角形，因此提出了一种算法，在保持墙体相同的情况下，尽量减少每个表面的三角形数量；

（3）圆柱形构件可以用提出的方法简化，去除所有多余的三角形；

（4）其他弯曲部件也可以用一种建议的方法来简化，这种方法可以在多边形减少率和变形之间取得平衡。

一个顶点折叠的示例：

![Untitled](Bim减面算法调研/Untitled%208.png)

> **冯雨晴,奚雪峰,崔志明.基于 WebGL 的 BIM 模型轻量化研究[J].苏州科技大学学报:自然科学版,2021,38(4):72-78.**

[基于 WebGL 的 BIM 模型轻量化研究.pdf](Bim减面算法调研/%E5%9F%BA%E4%BA%8EWebGL%E7%9A%84BIM%E6%A8%A1%E5%9E%8B%E8%BD%BB%E9%87%8F%E5%8C%96%E7%A0%94%E7%A9%B6.pdf)

这篇论文从 BIM 模型轻量化入手，提出利用 Revit API 的二次开发技术，主要步骤如下：

将模型进行轻量化压缩：模型数据结构建立-将 Revit 模型文件导出为 GLTF 文件，该文件是适用于各大 WebGL 引擎的模型文件格式

场景渲染优化：利用八叉树优化算法对模型场景进行管理，优化了大体积模型的渲染问题

场景展示：WebGL 的技术来解析 GLTF 格式文件

该论文所研究的内容并不是 BIM 模型的减面算法，但有可能成为优化场景的一个思路。
