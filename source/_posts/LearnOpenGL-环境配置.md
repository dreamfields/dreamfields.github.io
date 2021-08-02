---
title: LearnOpenGL-环境配置
tags:
  - OpenGL
categories:
  - - Configuration
date: 2021-08-02 11:50:58
---
# Step1：建立GLFW环境与配置GLAD

GLFW是一个专门针对OpenGL的C语言库，它提供了一些渲染物体所需的最低限度的接口。它允许用户创建OpenGL上下文，定义窗口参数以及处理用户输入。该步骤保证它恰当地创建OpenGL上下文并显示窗口。

GLAD是用来管理OpenGL的函数指针的，所以在调用任何OpenGL的函数之前我们需要初始化GLAD。


> 参考链接：
> - [Learn OpenGL——创建窗口](https://learnopengl-cn.github.io/01%20Getting%20started/02%20Creating%20a%20window/)
> - [openGL学习笔记七： glad库及使用](https://blog.csdn.net/u012278016/article/details/105582080)

