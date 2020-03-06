---
title: 人群计数论文解读之《CSRNet： Dilated Convolutional Neural Networks for Understanding the Highly Congested Scene》
tags: CrowdCounting
key : csrnet
pageview: true
sharing: true
comment: true
article_header:
  type: cover
  image:
    src: /cover/CSRNet.png

---

> ## 前言
>本篇博客介绍CVPR2018发表的论文 [CSRNet](http://www.zpascal.net/cvpr2018/Li_CSRNet_Dilated_Convolutional_CVPR_2018_paper.pdf)。
<!--more-->
推荐一个基于pytorch框架的简洁开源实现[Github](https://github.com/CommissarMa/CSRNet-pytorch)，建议对照学习。
注：正文包括笔者对原论文的翻译和自己的理解，请读者注意区分人称。

------

## Abstract
  本文提出了一种称为CSRNet的用于拥挤场景计数的网络，该方法可以理解高度拥挤的场景并进行准确的计数估计以及生成高质量密度图。我们提出的CSRNet由两个主要组件组成：卷积神经网络（CNN）作为2D特征提取的前端，而扩张卷积网络则作为后端，后者使用扩张卷积核提供更大的感受野并替换池化操作。  CSRNet由于具有纯卷积结构，因此是易于训练的模型。
  我们在四个数据集（ShanghaiTech数据集，UCF_CC_50数据集，WorldEXPO’10数据集和UCSD数据集）上验证了CSRNet，并体现了最优性能。在ShanghaiTech_Part_B数据集中，CSRNet的平均绝对误差（MAE）比以前的最新方法低47.3％。我们将应用领域扩展至其他目标计数，例如TRANCOS数据集中的车辆计数。结果表明，与以前的最新技术方法相比，CSRNet显着提高了输出质量，MAE降低了15.4％。


## Introduction
首先，文章点明了基于密度图估计的深度方法的挑战所在：1）MSE这一类逐像素回归的损失函数，会使得网络尽可能生成逐点平滑的密度图，这和实际情况往往不符合。2）不规则的人群聚集。3）变化的相机透视关系。
然后文章吐槽了一下MCNN。第一，MCNN训练时间比较长。第二，MCNN本意是通过各分支不同大小的卷积核
学习到不同尺度的信息，但实际上把训练好的MCNN的每一列拿出来测试，发现错误率曲线是非常相似的，说明每一列学到的信息实际上相同。

![图1 多列错误率曲线](/postimages/CSRNet/Multi_Column.png){:height="100%" width="100%"}

### Contributions of this paper

1. 使用全卷积网络，去掉所有pooling操作，避免特征图分辨率在Backbone之后继续降低：
2. 全部卷积核尺寸为3$\times $3
3. 使用空洞卷积作为网络后端。


## Related Work
略

## Method

### CSRNet 架构

使用VGG-16的前13层加一个$1\times 1$卷积作为前端backbone.

为了避免特征图分辨率在后端继续下降，使用空洞卷积作为后端。
#### 空洞卷积
空洞卷积看一张示意图就明白了：

![图2 空洞卷积示意图](/postimages/CSRNet/dilated_conv.png){:height="100%" width="100%"}

就是在普通的卷积核里面塞上空洞，这样把它变成更大的卷积核，然而实际运算量不变。（认为空洞不参与计算）

文章认为空洞卷积出来的特征图包含更多细节，如下图：

![图3 空洞卷积得出的特征图](/postimages/CSRNet/dilated_conv_feature.png){:height="100%" width="100%"}

### Optimization of MCNN
损失函数可以通过批随机梯度下降和反向传播进行优化。
但是，实际上，由于训练样本的数量非常有限，并且由于梯度消失现象的影响，因此同时学习所有参数并不容易。

受RBM预训练的影响，我们通过直接将第四卷积层的输出映射到密度图来分别对每个单列中的CNN进行预训练。
然后，我们使用这些经过预训练的CNN来初始化所有列中的CNN，并同时微调所有参数。

### Transfer learning setting

MCNN模型的一个优点是，学习到了不同头部大小的建模信息。
因此，如果在包含不同大小的头部的大型数据集上训练模型，则可以轻松地将模型迁移到其他头部具有某些特定大小的数据集。
如果目标域仅包含少量训练样本，则可以简单地在MCNN中固定每列中的前几层，并仅微调后几层卷积层。
在这种情况下，微调最后几层有两个优点。
首先，通过固定前几层，可以保留在源域中学习的知识，并且通过对后几层进行微调，可以使模型适应目标域。
因此，源领域和目标领域的知识都可以集成在一起，并有助于提高准确性。
其次，与微调整个网络相比，微调最后几层大大降低了计算复杂度。

## Experiments
具体实验结果省略，看一下发布的数据集Shanghaitech dataset吧。
### Shanghaitech dataset

我们引入了一个名为Shanghaitech的新的大规模人群计数数据集，其中包含1198幅带注释的图像，总共330,165人的头部中心注释。
据我们所知，就注释人的数量而言，该数据集是最大的数据集。
该数据集由两部分组成：A部分中有482张图像，是从Internet上随机抓取的，B部分中有716张图像，是从上海市的繁忙街道上拍摄的。
两个子集的人群密度存在显著变化，这使得人群的准确估计比大多数现有数据集更具挑战性。 
A部分和B部分都分为训练和测试：A部分的300张图像用于训练，其余182张图像用于测试； B部分的400张图像用于训练，316张用于测试。
表1给出了Shanghaitech数据集的统计数据及其与其他数据集的比较。

![数据集规模比较](/postimages/MCNN/datasetcomparison.png)

>另外还有一个比较有趣的实验是比较单列CNN与多列CNN的性能。

![单列CNN与MCNN](/postimages/MCNN/SingleCNN.png)
图6显示了Shanghaitech A上单列CNN与MCNN的比较。可以看出，对于MAE和MSE，MCNN明显优于每个单列CNN。这验证了MCNN结构的有效性。 
>这个实验验证的是将单列CNN分别单独训练的性能和MCNN使用这些单列CNN作为预训练的性能，以及MCNN从头开始训练的性能，可以看到几个单列CNN的性能是比较接近的(CSRNet的伏笔),MCNN性能总是优于单列CNN。预训练的MCNN优于从头开始训练的，这一点比较好理解，可以类比一下HourGlass网络中的中间监督，对网络每一个独立组件引入监督信息，使其变为一个集成学习器。

> MCNN是深度学习应用 于人群计数早期文章之一，至今仍然会被最新的论文在实验里引用23333,我们在阅读这类开创期论文的时候不要觉得它太简单，要注意其中出现的分析论断，在后续阅读其他论文时看一看哪些想法成为了领域主流，哪些想法被推翻，哪些想法还不够成熟，这样才能发掘出新的想法，（才能水论文）。

