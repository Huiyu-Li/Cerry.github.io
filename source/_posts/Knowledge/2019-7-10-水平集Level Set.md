---
title: 水平集-Level Set
date: 2019-07-10 21:00:00
author: Huiyu
summary: 水平集（Level Set）一种传统的图像分割方法。
img: /medias/pic_md/Knowledge/LevelSet1.png
mathjax: true
categories: Knowledge
tags: 
- Level Set
---
## 水平集（Level Set）
#### 曲线演化的直观解释
   假设$X\(t\)$表示一平面曲线，$T$表示切线，$N$表示法线。因为切向量$T$和法向量$N$互相垂直，所以平面上任何曲线都可以用曲线上任何一点的$T$和$N$的线性组合来表示：
$$\frac{\partial X}{\partial t} = \alpha T + \beta N$$
   如果只考虑几何形状的变化，则曲线变化只跟法线方向的变化有关系:
$$\frac{\partial X}{\partial t} = \beta N$$
   假设我们有一个3D surface和一个2D plane，如下所示，
![Level Set1](/medias/pic_md/Knowledge/LevelSet1.png)
   我们通过surface与plane的关系来描述curve（surface与plane的交线），通过调整surface来实现curve的变化，这就是level set的基本思想。
![Level Set2](/medias/pic_md/Knowledge/LevelSet2.png)

#### Level set 的数学定义及运动表示

   假设隐函数$φ\(X\(t\),t\)$表示一个高维空间的surface，程其在低维空间上的接触面为$φ\(X\(t\),t\)=0$
   我们按如下方式跟踪surface和curve的演化过程。
$$\frac{\partial X}{\partial t} = V(k)N$$
其中,$V(k)$称为**速度方程**，$k$表示曲率，$t$表示时间，$N =  - \frac{\nabla \phi }{\left\| {\nabla \phi } \right\|}$表示surface的内法线。
然后对surface对t进行级联求导，
$$\phi (X(t),t) = 0$$
$$\Rightarrow \nabla \phi {X_t} + {\phi _t} = 0$$
$$\Rightarrow {\phi _t} = V(k)\left\| {\nabla \phi } \right\|$$
   由此可知，给定初始的surface$ φ\(X\(t\),t\)$以及速度方程$V\(k\)$，便可以得到任意时刻的surface $φ$。

#### Level Set的数值解法
   熟知机器学习同学可能会发现，surface φ的演化过程酷似神经网络中的参数利用梯度下降更新过程。
$$\frac{\partial \phi }{\partial t} = \frac{\phi ^{t + \Delta t} - {\phi ^t}}{\Delta t}$$
$$\Rightarrow {\phi ^{t + \Delta t}} = {\phi ^t} + {\phi _t}\Delta t = {\phi ^t} + V(k)\left\| {\nabla \phi } \right\|\Delta t$$

参考：
[1] ["Level Set Method Part I: Introduction"](https://wiseodd.github.io/techblog/2016/11/05/levelset-method/)