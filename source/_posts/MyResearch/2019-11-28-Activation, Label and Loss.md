---
title: Activation and Loss
date: 2019-11-28 11:14:00
author: Huiyu
summary: 分析不同激活函数和Loss函数的配对。
mathjax: true
categories: MyResearch
tags: 
 - Segmentation
 - Regression
---
应用场景：医学图像的分割
分割结果由神经网络给出，所以，神经网络的最后一层决定了输出的特性。

## 激活函数

此处先介绍三组具有代表性的激活函数：

| $$Softmax(x_i) = \frac{exp(x_i)}{\sum\nolimits_jexp(x_j)}$$ | $$Sigmoid(x) = \frac{1}{1+exp(-x)}$$                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
|                                                             | <img src="/medias/pic_md/MyResearch/Sigmoid.png" width="200" hegiht="100" align=center /> |

| $$ReLU(x) = max(0,x)$$                                       | PReLU                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <img src="/medias/pic_md/MyResearch/Relu.jpg" width="200" hegiht="100" align=center /> | <img src="/medias/pic_md/MyResearch/PRelu.jpg" width="200" hegiht="100" align=center /> |

| Tanh                                                         | HardTanh                                                     | SoftShrink                                                   | TanhShrink                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| <img src="/medias/pic_md/MyResearch/Tanh.jpg" width="200" hegiht="100" align=center /> | <img src="/medias/pic_md/MyResearch/Hardtanh.png" width="200" hegiht="100" align=center/> | <img src="/medias/pic_md/MyResearch/SoftShrink.jpg" width="200" hegiht="100" align=center /> | <img src="/medias/pic_md/MyResearch/TanhShrink.jpg" width="200" hegiht="100" align=center /> |

# Loss
## Regression Segmentation
### 1. L1 Loss
$${L_1} = \left| {p - g} \right|$$
### 2. smooth L1 Loss
$$Smooth\ {L_1} = \left\{ {\begin{array}{*{20}{c}}
{0.5{x^2},\ \left|x \right| < 1}\\
{\left| x \right| - 0.5,\ x <  - 1\ or\ x > 1}
\end{array}} \right.$$

### 3. L2 loss
$${L_2} = {\left| {p - g} \right|^2}$$
### 4. Ridge regression
$$Rige = MSE(p,g;\theta ) + \alpha \frac{1}{2}\sum\nolimits_i {\theta _i^2} $$
### 5. LASSO
$$LASSO = MSE(p,g;\theta ) + \alpha \sum\nolimits_i {\left| {\theta _i} \right|}$$

参考：
[L1 loss VS L2 loss； L1 regularization VS L2 regularization](https://blog.csdn.net/weixin_44058333/article/details/103205940)

