---
layout:     post
title:      "第三届阿里安全赛0day总结"
subtitle:   " \"Alibaba Cloud's 3rd Annual Security Algorithm Challenge\""
date:       2018-09-24 17:54:00
author:     "FinlayLiu"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 数据挖掘
---

# 赛题分析

Fast RCNN也是来自RBG大神，也是对物体检测流程的改进，基于RCNN和SPP-Net的基础。Fast RCNN借鉴了SPP-Net的思路，简化了从proposal提取特征的次数，也对检测流程进行了改进。由于SPP-Net的精度和RCNN旗鼓相当，因此Fast RCNN很大篇幅介绍了提升的精度来自什么操作，也做了一系列实验。
![](/img/post/dl_fastrcnn1.png)


# 数据分析

论文针对SPP-Net和RCNN的检测流程进行改进，利用SPP-Net的思路对检测流程进行缩减。先提取proposal，然后通过整张图的CNN特征，得到目标边框特征。

Fast RCNN在确定物体类别上，并没有使用之前的SVM训练，而是使用了全连接FC层，并加入了边框位置的回归训练，进一步提高了性能和精度。作者还针对具体的网络结构以及具体的训练方法进行了分别实验比较，证明分类loss和边框回归loss是有效的。
![](/img/post/dl_fastrcnn2.png)

# 特征工程

# 构建模型

# 赛后思考
