---
layout: post
title:  "OpenGL: 摄像机类的设计(摄像机旋转的公式推导)"
date:   2016-12-01 20:16:00 +0800
categories: OpenGL
tags: 数学 图形学
---

* content
{:toc}
 
### **前言**
&emsp;&emsp;前一段时间在follow [LearOpenGL]() 学习OpenGL的核心模式，主要是学习GLSL和与GLSL配套的OpenGL。当我看到 [Camera](http://www.learnopengl.com/#!Getting-started/Camera) 这一章的时候，他在说明摄像机旋转的时候有说得不是很清楚(从原PO的下面评论区一堆人发问可以看出来)。其实原理是很简单的高中数学问题，在这和大家分享一下自己的思考过程。(原教程还是非常棒的，不想看原PO英文的同学推荐看中文版的教程[LearnOpenGL-CN](https://learnopengl-cn.github.io/01%20Getting%20started/09%20Camera/))

![pic](/images/question1.png)
![pic](/images/question2.png)

(下面的评论区很多人对这点发问)

### **问题描述**
&emsp;&emsp;要设计一个第一人称视角的摄像机，摄像机无非2种操作：平移(玩家位置移动)，旋转(玩家视角变化，比如向左看，像右看...)。我们单纯考虑摄像机的旋转问题(平移比较简单，有兴趣可以看原Po或者看我下面给出的实现)。任何编程问题我们都需要明确我们的输入和输出是什么，那么我们得到的输入是：鼠标在水平和竖直两个方向的平移量，我们需要输出的是：摄像机的朝向，我们用一个从原点出发的三维向量(x,y,z)表示。

### **思考过程**
#### **欧拉角:**
&emsp;&emsp;我们先介绍欧拉角的概念，欧拉根据旋转轴的不同把旋转分为3种不同的旋转，分别是：类似于点头动作Pitch，和类似以摇头动作的Yaw，以及类似于在地上滚来滚去Roll。
![pic](/images/eular.png)

&emsp;&emsp;结合一下我们玩过的FPS游戏，在**问题描述**中的输入，鼠标在两个方向的移动分量事实上可以看做是摄像机的Pitch角以及Yaw角。为什么没有Roll角？你特么会在游戏中做*歪头*的动作啊？！就连现实也很少做好伐？!

![pci](/images/head.jpg)

#### **小学数学：**
&emsp;&emsp;原文在这里出现了第一个难以理解的地方，原文的配图和描述是这样的：

![pic](/images/triangle.png)


> If we define the hypotenuse to be of length 1 we know from trigonometry (soh cah toa) that the adjacant side's length is cos x/h=cos x/1=cos xcos⁡ x/h=cos⁡ x/1=cos⁡ x and that the opposing side's length is sin y/h=sin y/1=sin ysin⁡ y/h=sin⁡ y/1=sin⁡ y. This gives us some general formulas for retrieving the length in both the x and y directions, depending on the given angle. 

&emsp;&emsp;我看完原文之后的表情和上面尔康是一样的(???)：为毛cosθ=x/h,然后直角边就变成了cosx啊？？？本来很简单的东西一下子就变懵逼了。我认为正确的理解应是这样：

![pic](/images/triangle_fixed.png)

#### **二维映射三维：**
