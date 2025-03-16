---
title: Cascade Net的训练
date: 2020-1-4
author: Huiyu
summary: 
categories: MyResearch
tags: 
 - PyTorch
---
本文主要分析Cascade Net（级联网络）的训练情况。
# 2个网络，2个loss，梯度如何回传？
![](/medias/pic_md/MyResearch/net1net2.png)
1. loss1只会优化net1;
2. loss2优化net1和net2。
3. 推荐使用1个优化器，优化一个loss(loss1+loss2), 方便简洁.

公共代码部分：
~~~python
import torch
from torch import nn
import numpy as np

class Net1(nn.Module):
    def __init__(self):
        super().__init__()
        self.layer1 = nn.Sequential(
            nn.ConvTranspose3d(1, 1, kernel_size=2),
            nn.InstanceNorm3d(1),
            nn.ReLU(inplace=True)
        )
    def forward(self, input):
        layer1 = self.layer1(input)
        return layer1
class Net2(nn.Module):
    def __init__(self):
        super().__init__()
        self.layer2 = nn.Sequential(
            nn.ConvTranspose3d(1, 1, kernel_size=2),
            nn.InstanceNorm3d(1),
            nn.ReLU(inplace=True)
        )
    def forward(self, input):
        layer1 = self.layer2(input)
        return layer1

loss1 = nn.SmoothL1Loss()
loss2 = nn.SmoothL1Loss()

net1 = Net1()
net2 = Net2()

input = torch.ones([1,1,2,2,2])
target1 = torch.ones([1,1,3,3,3])
target2 = torch.ones([1,1,4,4,4])
~~~
### 2个优化器独立优化2个Net，两个loss独立回传
~~~python
optimizer1.zero_grad()
loss_output1.backward(retain_graph=True)
# optimizer1.step()
for name,param in net1.named_parameters():
    print(name,param.grad)
for name,param in net2.named_parameters():
    print(name,param.grad)
    
optimizer2.zero_grad()
loss_output2.backward()
# optimizer2.step()
for name,param in net1.named_parameters():
    print(name,param.grad)
for name,param in net2.named_parameters():
    print(name,param.grad)
~~~
output:
> layer1.0.weight tensor([[[[[ 0.1047, -0.0668],
>                     [ 0.1183,  0.0265]],
>                     [[ 0.0277,  0.1389],
>                     [-0.0696,  0.1308]]]]])
> layer1.0.bias tensor([-4.6566e-09])
> layer2.0.weight None
> layer2.0.bias None
>
> layer1.0.weight tensor([[[[[ 0.1112, -0.0436],
>                     [ 0.0883,  0.0262]],
>                     [[ 0.0384,  0.1336],
>                     [-0.1178,  0.1256]]]]])
> layer1.0.bias tensor([-2.7940e-09])
> layer2.0.weight tensor([[[[[ 0.0079,  0.0407],
>                     [-0.0628,  0.0299]],
>                     [[-0.0284, -0.0627],
>                     [ 0.0145,  0.0101]]]]])
> layer2.0.bias tensor([1.1642e-09])
### 1个优化器优化2个Net
~~~python
optimizer1 = torch.optim.Adam([{'params': net1.parameters()},{'params': net2.parameters(),}], lr=0.001)
~~~
### 1个优化器优化2个Net，2个loss加和回传
~~~python
optimizer = torch.optim.Adam([{'params':net1.parameters()},{'params':net2.parameters()}], lr=0.001)

net1.train()
net2.train()
output1 = net1(input)
output2 = net2(output1)
loss_output1 = loss1(output1, target1)
loss_output2 = loss2(output2, target2)
loss_output = loss_output1+loss_output2

optimizer.zero_grad()
loss_output1.backward(retain_graph=True)
# optimizer1.step()
for name,param in net1.named_parameters():
    print(name,param.grad)
for name,param in net2.named_parameters():
    print(name,param.grad)

optimizer.zero_grad()
loss_output2.backward(retain_graph=True)
# optimizer2.step()
for name,param in net1.named_parameters():
    print(name,param.grad)
for name,param in net2.named_parameters():
    print(name,param.grad)

optimizer.zero_grad()
loss_output.backward()
# optimizer2.step()
for name,param in net1.named_parameters():
    print(name,param.grad)
for name,param in net2.named_parameters():
    print(name,param.grad)
~~~
output:
> layer1.0.weight tensor([[[[[ 0.0281,  0.0501],
>                     [-0.0897,  0.1292]],
>                     [[ 0.1451,  0.0634],
>                     [-0.0040,  0.0151]]]]])
> layer1.0.bias tensor([9.9535e-09])
> layer2.0.weight None
> layer2.0.bias None
> 
> layer1.0.weight tensor([[[[[ 0.0686,  0.0366],
>                     [ 0.0078,  0.0646]],
>                     [[ 0.0461,  0.0027],
>                     [-0.0182, -0.0114]]]]])
> layer1.0.bias tensor([-1.6298e-09])
> layer2.0.weight tensor([[[[[0.3400, 0.2140],
>                     [0.2509, 0.2214]],
>                     [[0.3691, 0.2921],
>                     [0.3126, 0.2221]]]]])
> layer2.0.bias tensor([-6.5425e-08])
> 
> layer1.0.weight tensor([[[[[ 0.0966,  0.0867],
>                     [-0.0819,  0.1939]],
>                     [[ 0.1912,  0.0661],
>                     [-0.0222,  0.0037]]]]])
> layer1.0.bias tensor([-6.0536e-09])
> layer2.0.weight tensor([[[[[0.3400, 0.2140],
>                     [0.2509, 0.2214]],
>                     [[0.3691, 0.2921],
>                     [0.3126, 0.2221]]]]])
> layer2.0.bias tensor([-6.5425e-08])
### 冻结net2的参数
~~~python
for param in net2.parameters():
    param.requires_grad = False
optimizer1 = torch.optim.Adam([{'params': net1.parameters()},{'params': filter(lambda p: p.requires_grad, net2.parameters()),}], lr=0.001)
# optimizer1 = torch.optim.Adam(net1.parameters(), lr=0.001)# since no param of frozen net2 need to be optimized, so this one is equal to the above optimizer.
~~~
output:
> layer1.0.weight tensor([[[[[ 0.1854,  0.0710],
>                     [-0.0503, -0.0730]],
>                     [[ 0.0512,  0.0873],
>                     [-0.1704,  0.0965]]]]])
> layer1.0.bias tensor([-2.2352e-08])
> layer2.0.weight None
> layer2.0.bias None
> 
> layer1.0.weight tensor([[[[[ 0.2990,  0.1330],
>                     [-0.2335, -0.1387]],
>                     [[-0.1399,  0.1262],
>                     [-0.2014,  0.2233]]]]])
> layer1.0.bias tensor([-6.7055e-08])
> layer2.0.weight None
> layer2.0.bias None
# 1个网络，2个loss，梯度如何回传？
![](/medias/pic_md/MyResearch/net.png)
1. loss1只会优化net1;
2. loss2优化net1和net2。
3. loss1+loss2一起回传也遵循以上原则。

公共代码部分：
~~~python
import torch
from torch import nn
import numpy as np

class Net(nn.Module):
    def __init__(self):
        super().__init__()
        self.layer1 = nn.Sequential(
            nn.ConvTranspose3d(1, 1, kernel_size=2),
            nn.InstanceNorm3d(1),
            nn.ReLU(inplace=True)
        )
        self.layer2 = nn.Sequential(
            nn.ConvTranspose3d(1, 1, kernel_size=2),
            nn.InstanceNorm3d(1),
            nn.ReLU(inplace=True)
        )
    def forward(self, input):
        layer1 = self.layer1(input)
        layer2 = self.layer2(layer1)
        return layer1,layer2

loss1 = nn.SmoothL1Loss()
loss2 = nn.SmoothL1Loss()

net = Net()

input = torch.ones([1,1,2,2,2])
target1 = torch.ones([1,1,3,3,3])
target2 = torch.ones([1,1,4,4,4])

optimizer = torch.optim.Adam(net.parameters(), lr=0.001)
~~~
### 2个loss单独回传
~~~python
net.train()
output1,output2 = net(input)
loss_output1 = loss1(output1, target1)
loss_output2 = loss2(output2, target2)

optimizer.zero_grad()
loss_output1.backward(retain_graph=True)
# optimizer1.step()
for name,param in net.named_parameters():
    print(name,param.grad)

optimizer.zero_grad()
loss_output2.backward()
# optimizer2.step()
for name,param in net.named_parameters():
    print(name,param.grad)
~~~
output:
> layer1.0.weight tensor([[[[[ 0.1199,  0.1322],
>                     [ 0.3313,  0.2889]],
>                     [[-0.2830,  0.0982],
>                     [ 0.2318,  0.4600]]]]])
> layer1.0.bias tensor([-5.2154e-08])
> layer2.0.weight None
> layer2.0.bias None
> 
> layer1.0.weight tensor([[[[[-0.1710,  0.0898],
>                     [ 0.0150, -0.0648]],
>                     [[ 0.0969,  0.2731],
>                     [ 0.1348,  0.1892]]]]])
> layer1.0.bias tensor([-5.4948e-08])
> layer2.0.weight tensor([[[[[0.0403, 0.0771],
>                     [0.2339, 0.4653]],
>                     [[0.2246, 0.4523],
>                     [0.1780, 0.3113]]]]])
> layer2.0.bias tensor([-4.0978e-08])
### 2个loss加和一起回传
~~~python
net.train()
output1,output2 = net(input)
loss_output1 = loss1(output1, target1)
loss_output2 = loss2(output2, target2)
loss_output = loss_output1+loss_output2

optimizer.zero_grad()
loss_output1.backward(retain_graph=True)
# optimizer1.step()
for name,param in net.named_parameters():
    print(name,param.grad)

optimizer.zero_grad()
loss_output2.backward(retain_graph=True)
# optimizer2.step()
for name,param in net.named_parameters():
    print(name,param.grad)

optimizer.zero_grad()
loss_output.backward()
# optimizer2.step()
for name,param in net.named_parameters():
    print(name,param.grad)
~~~
output:
> layer1.0.weight tensor([[[[[ 0.0622,  0.1044],
>                     [ 0.0046,  0.0056]],
>                     [[ 0.0011, -0.1298],
>                     [ 0.0423, -0.0172]]]]])
> layer1.0.bias tensor([-1.8626e-09])
> layer2.0.weight None
> layer2.0.bias None
>
> layer1.0.weight tensor([[[[[ 0.0438,  0.0806],
>                     [ 0.0495, -0.0116]],
>                     [[ 0.0808, -0.0380],
>                     [ 0.0780,  0.0077]]]]])
> layer1.0.bias tensor([1.1176e-08])
> layer2.0.weight tensor([[[[[ 0.2026,  0.1759],
>                     [-0.0217,  0.1291]],
>                     [[ 0.0584,  0.1224],
>                     [ 0.0403,  0.1459]]]]])
> layer2.0.bias tensor([3.2596e-09])
>
> layer1.0.weight tensor([[[[[ 0.1061,  0.1850],
>                     [ 0.0541, -0.0060]],
>                     [[ 0.0819, -0.1678],
>                     [ 0.1203, -0.0095]]]]])
> layer1.0.bias tensor([3.7253e-09])
> layer2.0.weight tensor([[[[[ 0.2026,  0.1759],
>                     [-0.0217,  0.1291]],
>                     [[ 0.0584,  0.1224],
>                     [ 0.0403,  0.1459]]]]])
> layer2.0.bias tensor([3.2596e-09])
# 综上所述：
使用cascade net+2个loss, 为了方便单独load已经训练好的模型或者冻结某个网络的参数，推荐使用2个单独的网络，1个优化器，2个loss加和一起回传。
~~~python
import torch
from torch import nn
import numpy as np

class Net1(nn.Module):
    def __init__(self):
        super().__init__()
        self.layer1 = nn.Sequential(
            nn.ConvTranspose3d(1, 1, kernel_size=2),
            nn.InstanceNorm3d(1),
            nn.ReLU(inplace=True)
        )
    def forward(self, input):
        layer1 = self.layer1(input)
        return layer1
class Net2(nn.Module):
    def __init__(self):
        super().__init__()
        self.layer2 = nn.Sequential(
            nn.ConvTranspose3d(1, 1, kernel_size=2),
            nn.InstanceNorm3d(1),
            nn.ReLU(inplace=True)
        )
    def forward(self, input):
        layer1 = self.layer2(input)
        return layer1

loss1 = nn.SmoothL1Loss()
loss2 = nn.SmoothL1Loss()

net1 = Net1()
net2 = Net2()

input = torch.ones([1,1,2,2,2])
target1 = torch.ones([1,1,3,3,3])
target2 = torch.ones([1,1,4,4,4])

for param in net2.parameters():
    param.requires_grad = False
optimizer = torch.optim.Adam(net1.parameters(), lr=0.001)# since no param of frozen net2 need to be optimized

net1.train()
net2.train()
output1 = net1(input)
output2 = net2(output1)
loss_output1 = loss1(output1, target1)
loss_output2 = loss2(output2, target2)
loss_output = loss_output1+loss_output2

optimizer.zero_grad()
loss_output.backward()
optimizer.step()
~~~