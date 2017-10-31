layout:     post
title:      " Kaggle - Allstate Claims Severity 总结"
subtitle:   " \" Kaggle - Allstate Claims Severity Note\""
date:       2016-12-23 12:00:00
author:     "Finlay"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 数据挖掘
---

Allstate最终PB结果75，在意料之中。整体上说来，这个比赛还是很有意思比较靠谱，可以学到很多姿势。

在这次比赛学到新工具：
- Keras调参
- RGF
- lightGBM

这次比赛还有没做好的地方：
- 数据没分析好，导致kernel结果都一样。
- 本地CV没做好记录，完全相信PB被坑。
- 没有做stacking，重大失误。

总结积累的经验：
1. 一定要做stacking，从开始就要不停尝试新方法。
2. 文档一定要写好，记录完整，做ensemble时候用。
3. 相信本地CV和融合CV，PB不靠谱。
4. 人工调整XGB参数费时间，而且存在盲区。

[Bishwarup B #1st Place Solution](https://www.kaggle.com/c/allstate-claims-severity/forums/t/26416/1st-place-solution)

[Alexey Noskov #2nd Place Solution](https://www.kaggle.com/c/allstate-claims-severity/forums/t/26427/2nd-place-solution)
https://github.com/alno/kaggle-allstate-claims-severity

[Faron's 3rd Place Solution](https://www.kaggle.com/c/allstate-claims-severity/forums/t/26447/faron-s-3rd-place-solution)
