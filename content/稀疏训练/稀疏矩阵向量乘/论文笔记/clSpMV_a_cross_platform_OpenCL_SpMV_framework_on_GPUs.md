---
linktitle: clSpMV.. a cross-platform OpenCL SpMV framework on GPUs
summary: Summary of clSpMV.. a cross-platform OpenCL SpMV framework on GPUs
type: book
weight: 2
---
# clSpMV: a cross-platform OpenCL SpMV framework on GPUs
基本思想：不同的稀疏格式，在不同的硬件和不同的稀疏拓扑结构下，有不同的表现。所以可以考虑将稀疏矩阵分解为多个子矩阵，为每个子矩阵使用不同的稀疏格式。
## 鸡尾酒格式
混合多种稀疏格式的数据格式，支持了九种格式：
![](clSpMV_a_cross_platform_OpenCL_SpMV_framework_on_GPUs-1662189891833.jpeg)