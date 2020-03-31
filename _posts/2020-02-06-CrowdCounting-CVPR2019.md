---
title: 人群计数CVPR2019论文综述
tags: CrowdCounting
key : cvpr2019survey
pageview: true
sharing: true
comment: false

---

> ## 前言
>最近组会做了一个人群计数CVPR2019的综述，分享一下。
<!--more-->

------

## 概要
  CVPR2019一共录用了12篇人群计数方向的论文，可以说是很热门的领域。我将这12篇论文的改进划分为一下几类：
  - 网络结构 Network
    - 多尺度信息融合 Multi-Scale fusion
    - 注意力机制 Attention Mechanism
    - 定位信息 Localization
    - 透视关系 Perspective
  - 数据 Data and Unsupervised Learning
  - 损失函数 Loss Function

下面我依次简要介绍这12篇论文，具体的论文解读请看后续博客。

## 网络改进

### 多尺度信息融合

#### TEDNet (Trellis Encoder-Decoder Networks)

采用Encoder-Decoder架构，其中Encoder部分模仿Inception, 使用多个尺寸卷积核进行并联，Decoder部分将相邻分辨率特征图进行融合。

#### Context-Aware Crowd Counting

通过不同程度的平均池化获取不同尺度的上下文信息，并通过卷积映射为不同尺度下的特征图权重，对原特征图加权。

### 注意力机制

#### Residual Regression with Semantic Prior for Crowd Counting

通过聚类算法，将与输入图片密度信息相似的支撑图片一同输入到网络中，学习出支撑图片与输入图片的密度图残差，再将残差加到支撑图片的密度图上，相当于做了投票（或者集成学习）。

#### ADCrowdNet: An Attention-injective Deformable Convolutional Network for Crowd Understanding

先用一个完整的网络（front-end + backend) 生成输入图像的attention map，代表拥挤情况，将attention map 与原图点乘，再经过另一个完整网络生成最终密度图，后者使用了可变形卷积。

#### Recurrent Attentive Zooming for Joint Crowd Counting and Precise Localization

一方面用localization map和密度图一起监督，另一方面生成attention map 然后用 RPN 生成proposals,对这些proposals进行放大，达到精准计数的目的。

### 定位
#### Point in, Box out
迭代生成精准的人头框

### 透视关系

#### Revisiting Perspective Information for Efficient Crowd Counting

训练一个网络生成透视图，将透视图融合到密度图预测网络中。

#### Leveraging Heterogeneous Auxiliary Tasks to Assist Crowd Counting

采用多任务框架，三个任务：预测count数量，预测深度图（GT是通过一个预训练网络生成的），0-1 mask分割（GT是通过密度图GT设置阈值得到的）。

## 数据
#### Leveraging Heterogeneous Auxiliary Tasks to Assist Crowd Counting

用GTA5游戏引擎生成仿真数据增强数据集。