---
title: CRF as RNN
date: 2019-07-15 21:00:00
author: Huiyu
summary: 用RNN实现CRF。
img: /medias/pic_md/Knowledge/CRFasRNN1.jpg
mathjax: true
categories: Knowledge
tags:
 - CRF
 - RNN
---
**条件随机场(CRF )的概率函数**为$P(X = x|I) = \frac{1}{z}\exp ( - E(x|I))$

**CRF 的能量函数为**$E(x) = \sum\limits_i{\psi\_u}({x\_i})+\sum\limits_{i<j}{\psi\_p}({x\_i},{x\_j})$
其中第一项为数据项，第二项为平滑项，其定义为若干个高斯函数的和，如下公式所示。数据项约束每个像素尽可能分类正确，平滑项约束相邻像素之间的灰度值差异要尽可能小。
$$
{\psi_p}({x_i},{x_j}) = u({x_i},{x_j})\sum\limits_{m=1}^M{\omega ^{(m)}}{k^{(m)}}({f_i},{f_j}))
$$
![](/medias/pic_md/Knowledge/CRFasRNN3.png)
![ A mean-field iteration as a CNN]( /medias/pic_md/Knowledge/CRFasRNN2.jpg)
参考：
[CRF as RNN的原理及Caffe实现](https://blog.csdn.net/taigw/article/details/51794283)
[2015_Conditional Random Fields as Recurrent Neural Networks_ICCV](https://www.cv-foundation.org/openaccess/content_iccv_2015/html/Zheng_Conditional_Random_Fields_ICCV_2015_paper.html)