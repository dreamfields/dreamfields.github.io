---
title: Games202-01前言
tags:
  - CG
categories:
  - - CG
    - GAMES202
  - - Theory
date: 2022-11-01 01:21:05
---

# 课程内容总览

### **重要的算法**

实时阴影：PCF、PCSS

实时环境映射：shading from environment lighting（此时不考虑环境光中的阴影）

实时环境/全局光照：PRT（准确的得到来自环境光中的阴影，分为 diffuse 和 glossy）

实时全局光照（其实就是直接+间接光照）：

- 图像空间的 GI：RSM
- 3D 空间的 GI：LPV、VXGI
- 屏幕空间的 GI：SSAO、HBAO（Horizon Based ambient occlusion）、SSDO、SSR

光线追踪降噪技术：SVGF

> 基于图像的全局光照（Image based illumination）。**IBL**是基于**物理渲染的真实感**的重要来源，是对**环境光照**的一种处理方案。对于大部分情况来说，环境光来自于**天空盒**，也就是**cube-map 贴图**。因此，IBL 的重点就在于如何从图像中获取光照信息。

# 实时渲染领域(RTR4)

RTR4 中值得看的新增章节是如下四章：

- Chapter 9 Physically Based Shading 基于物理的着色
- Chapter 20 Efficient Shading 高效率着色
- Chapter 21 Virtual and Augmented Reality 虚拟现实和增强现实
- Chapter 26 Real-Time Ray Tracing 实时光线追踪


# 参考

- [GAMES202-高质量实时渲染\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1YK4y1T7yY?p=1)
- [Games202 高质量实时渲染课堂笔记](https://zhuanlan.zhihu.com/p/363333150)
- [【论文复现】Reflective Shadow Maps - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/357259069)
- [Spatiotemporal Variance-Guided Filter（实时光线追踪）- 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/28288053)
- [知乎-毛星云-如何评价《Real–time Rendering》第四版?](https://www.zhihu.com/question/290566100/answer/471199400)
- [实时渲染第四版-目录](https://zhuanlan.zhihu.com/p/406606440)

- [实时渲染第四版学习笔记](https://www.zhihu.com/column/c_1221792809822347264)

- [ShineEngine](http://geekfaner.com/shineengine/translate.html)
