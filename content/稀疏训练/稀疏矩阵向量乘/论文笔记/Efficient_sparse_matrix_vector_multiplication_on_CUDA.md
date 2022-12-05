---
linktitle: Efficient sparse matrix-vector multiplication on CUDA
summary: Summary of Efficient sparse matrix-vector multiplication on CUDA
type: book
weight: 1
---
# Efficient sparse matrix-vector multiplication on CUDA
比较早的工作，使用cuda来加速稀疏矩阵向量乘（SpMV）
论文：**Implementing Sparse Matrix-Vector Multiplication on Throughput-Oriented Processors**
对应的技术报告：**Efficient sparse matrix-vector multiplication on CUDA**

## 1.稀疏格式
### DIA
### ELL
### CSR
### COO


## 2.SpMV算法
下面的代码都是论文里经过简化后的代码，用来表达核心思想。实际工程代码会考虑更多细节。
### DIA格式下的SpMV实现

![](稀疏矩阵存储格式-1662113858029.jpeg)

### ELL格式下的SpMV实现
![](稀疏矩阵存储格式-1662117591836.jpeg)

### CSR
#### CPU版本
![](Efficient_sparse_matrix_vector_multiplication_on_CUDA-1662118602466.jpeg)
#### GPU版本(scalar kernel)
![](Efficient_sparse_matrix_vector_multiplication_on_CUDA-1662118673592.jpeg)
#### GPU版本(vector kernel)
![](Efficient_sparse_matrix_vector_multiplication_on_CUDA-1662124562548.jpeg)

### COO 
关键步骤：分段规约（完整的代码中，怎么用到这个了？）
![](Efficient_sparse_matrix_vector_multiplication_on_CUDA-1662127991277.jpeg)

