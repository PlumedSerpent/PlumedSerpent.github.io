---
title: 人群计数论文解读之《Switching Convolutional Neural Network for Crowd Counting》
tags: [CrowdCounting,Switch-CNN]
key : switch-cnn
pageview: true
sharing: true
comment: true
article_header:
  type: cover
  image:
    src: /cover/SWITCH-CNN.png

---

> ## 前言
>本篇博客介绍CVPR2017发表的论文 [Switch-CNN论文地址](http://openaccess.thecvf.com/content_cvpr_2017/papers/Sam_Switching_Convolutional_Neural_CVPR_2017_paper.pdf)。[官方代码地址](https://github.com/val-iisc/crowd-counting-scnn)
<!--more-->
本文依旧是采用多个不同尺度的CNN来处理不同密度的图像的思路，不同的是采用一个CNN网络来进行尺度的抉择。
------

## Abstract
  本文提出了一种新的人群计数模型，实现了从密集人群图像到其密度分布的映射。密集人群计数问题的难点包括人头互相遮盖、人群与背景相似度高以及相机视角的较大差异等，此前效果较好的人群计数网络使用了多尺寸CNN、循环网络或多列CNN特征融合的方法来处理这些问题。本文提出了切换式卷积神经网络switching convolutional neural network（Switch-CNN）来提升人群计数的精确度，根据训练的CNN获取人群密度，将人群场景中的patch分发到独立的CNN回归器，将其结果作为最终结果。

## Introduction
略。

## Related Work

略。



## Approach
之前的工作通过使用具有特征融合技术的多列CNN架构来回归人群密度，对传统的卷积架构进行了修改，以对密集人群中尺度引起的极端变化进行建模。

在本文中，我们考虑了切换式CNN结构（Switch-CNN），该结构可以分发patch至不同的回归器。与MCNN网络一样，这些独立的回归器感受野不同。切换分类器与多个CNN回归器交替训练，以正确地将补丁分发到特定回归器。与MCNN不同的是，MCNN对于多尺度特征融合是固定的全局的，而不是本文这样自适应的，局部的。

### 网络结构

输入图像被切分成不重叠的9个patch，这样可以近似认为每个patch的密度是一致的。Switch-CNN的网络结构由一个切换分类器和3个回归器组成。如图1所示。
![图1 Switch-CNN网络结构](/postimages/Switch/arch.png)

>关于这个网络结构，有一个令人疑惑的地方是这个Switch Layer到底是什么东西。我check了一下官方代码，感觉就是一个argmin用来选择loss反传？其余部分的结构可参见原文，没什么值得特别说明的。

### GT生成
与MCNN相同。
整个网络训练分为三个过程，预训练（Pretraining）、差异训练（differential training）、耦合训练（coupled training）。

### 预训练

独立地预训练三个回归器，用L2 Loss，每个回归器都用全部数据训练。

### 差异训练
预训练使得每个回归器尽可能拟合所有密度的patch，但这是不可能的，所以差异训练是给定每一个patch，只对绝对误差也就是L1 Loss最小的那个回归器进行更新，使得这些回归器产生对patch密度的bias。

### 耦合训练

这个阶段主要是为了训练切换分类器。又分为两个步骤。切换器训练和switched differential training。具体来讲，首先根据差异训练得到的回归器，生成切换分类器所需要的label，对每一个patch, 如果回归器i的预测误差最小，那么该patch的密度分类为i。但是这样的label对与分类器很难学习（因为所谓不同回归器负责不同密度的patch是我们一厢情愿的假设，实际上每个回归器究竟对哪些样本有bias是我们不知道的，也是分类器不知道的，因此很难分类。因而，本文提出让回归器和分类器一起训练，也就是根据分类器的结果，将给定patch分发给分类器所决定的回归器，并根据loss更新这个回归器。如下图算法图所示：在每一个epoch里，先独立训练分类器一轮（注意是一轮而不是一个step），再根据训练之后的分类器的预测结果分发patch，进而计算loss，更新对应的回归器。然后回归器重新产生密度的分类label，进入下一轮。周而复始。

![训练算法](/postimages/Switch/algo.png)

另外为了克服样本不均衡，对分类样本少的类别，随机采样多次，使得数量均衡，看代码：

``` Python
num_files_per_class = max([len(ds) for ds in data])
#获取样本最多的类别的样本数
train_data = []
for i, ds in enumerate(data):
#遍历每一类别
    samples = []
    samples += ds
    while len(samples) < num_files_per_class:
    #随机采样将数量差异补齐
        samples += random.sample(ds,
            min(num_files_per_class - len(samples), len(ds)))
    random.shuffle(samples)                 
    train_data += samples
```

## Experiments
略。