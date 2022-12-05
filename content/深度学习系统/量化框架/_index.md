---
# Title, summary, and page position.
title: 量化框架
linktitle: 量化框架
summary: 量化框架
weight: 1
# Page metadata.
date: '2018-09-09T00:00:00Z'
type: book # Do not modify.
toc: false
---

1.各层有多少卷积层输入数据超过范围？

2.qi.zeropoint,qo.zeropoint,scale的原理是什么？

3.为什么减去qi.zeropoint后会超出范围？

4.为什么对称量化训不上去？

5.用10%精度的数据导出数据流试试，看会不会分布很不正常？

6.数据流加入对卷积输入特征图和权重的大小的检查功能

7.在ACT上尝试跑多卡

8.学pytorch allreduce原语

9.评估量化框架能否搞成数据并行

{{< list_children >}}