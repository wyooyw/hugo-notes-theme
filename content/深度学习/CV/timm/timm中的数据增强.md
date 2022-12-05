---
linktitle: timm中的数据增强
summary: Summary of timm中的数据增强
type: book
---
# timm中的数据增强

https://zhuanlan.zhihu.com/p/430563265

https://readpaper.com/paper/3201822691

## 1.MixUp

## 1.1 算法

### 1.1.1 Mixup

论文： mixup: Beyond Empirical Risk Minimization

地址： https://readpaper.com/paper/2765407302

作者：from Meta

发表：ICLR

### 1.1.2 Cutmix


方法：把另一张图挖一块放到当前图片里，同时标签也按比例混合，如下图最右侧所示。
![](timm中的数据增强-1665483174799.jpeg)


## 1.2 timm中的使用与实现

论文：CutMix: Regularization Strategy to Train Strong Classifiers with Localizable Features

地址：https://readpaper.com/paper/2944223741

作者：from Meta

发表：
data/mixup.py中的Mixup类

| 配置                                  | 用途                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| mixup_alpha（args.mixup）             | mixup论文中的参数$\alpha$，$\lambda=beta(\alpha,\alpha)$     |
| cutmix_alpha（args.cutmix）           | cutmix论文中的参数$\alpha$，$\lambda=beta(\alpha,\alpha)$    |
| cutmix_minmax（args.cutmix_minmax）   | cutmix中的参数，不知道干啥用的                               |
| prob（args.mixup_prob）               | 启动mixup和cutmix的概率                                      |
| switch_prob（args.mixup_switch_prob） | 当同时启动了mixup和cutmix时，使用cutmix的概率                |
| mode（args.mixup_mode）               | 可选：'batch', 'pair', 'elem' 。看代码，这仨干的是同一件事？？ |
| label_smoothing（args.smoothing）     | 用于标签平滑（例如[1,0,0,0]变为[0.9,0.1,0.1,0.1]这种）       |
| num_classes（args.num_classes）       | 分类数，对label做mix的时候会用到                             |

