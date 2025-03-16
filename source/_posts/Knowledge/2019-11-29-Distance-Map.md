---
title: Distance Transform
date: 2019-11-29
author: Huiyu
summary: Matlab 和 Python 计算Distance Transform。
categories: Knowledge
tags: 
- Distance Transform
- Matlab 
- Python
---
写在前面：近几天研究处于卡壳状态，疯狂地Coding，无奈的失败。。。
感觉老师说得特别对。一项研究开始之前，要不断地思考，先明确它的物理意义，然后再去动手实现。而不是像我这样，不管三七二十一，先去实现，结果不好，废铜烂铁。
一场有深度，有价值的思考，可以给自己接下来的实验省去许多不必要的麻烦。一个新的想法，如果理论上说不通，基本上没什么存活的意义。遇事多问为什么？
我为什么要这这样做？
它的作用是什么？
它的物理意义是什么？

# Distance Transform
先了解一下什么是Distance Transform？它的物理意是什么？
[Distance transform(距离变换)](https://blog.csdn.net/weixin_44058333/article/details/100186066)
# Matlab 计算Distance Transform
bwdist()会将前景置零，计算背景像素点的Distance Transform。
~~~malab
input = [[0,1,1,1,1,0];
         [0,1,1,1,1,0];
         [0,0,0,1,1,0];
         [0,0,0,1,1,0];
         [0,0,0,1,1,0]];
output = bwdist(input);
disp(output);
~~~
output：默认计算背景的Distance Transform
>1.0000         0         0         0         0    1.0000
>1.0000         0         0         0         0    1.0000
>1.4142    1.0000    1.0000         0         0    1.0000
>2.2361    2.0000    1.0000         0         0    1.0000
>3.0000    2.0000    1.0000         0         0    1.0000
# Python 计算Distance Transform
[distance_transform_edt](https://docs.scipy.org/doc/scipy-0.14.0/reference/generated/scipy.ndimage.morphology.distance_transform_edt.html)会将背景置零，计算前景的Distance Transform（和matlab的计算结果正好相反）。使用np.logical_not()可以得到前景的Distance Transform。
~~~pytho
from scipy.ndimage import distance_transform_edt
import numpy as np
import matplotlib.pyplot as plt
input = np.array([[0,1,1,1,1,0],
                   [0,1,1,1,1,0],
                   [0,0,0,1,1,0],
                   [0,0,0,1,1,0],
                   [0,0,0,1,1,0]])
out0 = distance_transform_edt(np.logical_not(input))
out1 = distance_transform_edt(input)
print(out0)
print(out1)
~~~
out0:背景的Distance Transform
>[[1.         0.         0.         0.         0.         1.        ]
> [1.         0.         0.         0.         0.         1.        ]
> [1.41421356 1.         1.         0.         0.         1.        ]
> [2.23606798 2.         1.         0.         0.         1.        ]
> [3.         2.         1.         0.         0.         1.        ]]

out1:默认计算前景的Distance Transform
>[[0.         1.         2.         2.         1.         0.        ]
>[0.         1.         1.         1.41421356 1.         0.        ]
>[0.         0.         0.         1.         1.         0.        ]
>[0.         0.         0.         1.         1.         0.        ]
>[0.         0.         0.         1.         1.         0.        ]]

out0+out1

>[[1.         1.         2.         2.         1.         1.        ]
> [1.         1.         1.         1.41421356 1.         1.        ]
> [1.41421356 1.         1.         1.         1.         1.        ]
> [2.23606798 2.         1.         1.         1.         1.        ]
> [3.         2.         1.         1.         1.         1.        ]]

| ![binaryMap](/medias/pic_md/Knowledge/DistanceMap2.png) | ![binaryMap](/medias/pic_md/Knowledge/DistanceMap3.png) |
| ------------------------------------------------------- | ------------------------------------------------------- |
|                                                         |                                                         |

综上，前景和背景的Distance Transform不可共存，到底选用哪一款，当然看你的实验需要。