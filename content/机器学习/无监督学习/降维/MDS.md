MDS（Multiple Dimensional Scaling，多维缩放）

### 概念：
MDS是用来降维的，并保持任意两个样本间的距离不变。假设原本每个样本用d维向量表示，经过MDS算法后，每个样本只需要用d'维向量来表示（d'<d），维度降低了（降维）；同时，任意两个样本i和j，如果原来它们的距离是dist(i,j)，则经过MDS算法降维后，它们的距离还是dist(i,j)，即保持距离不变。

上面是理想情况，很多时候无法保证距离严格相等，只能近似相等。

### 算法：

见西瓜书P227-P229

一些理解：
- 距离精度的损失，发生在“取前k大的特征值”的时候，有点像截断SVD（或许是等价的？）
- 可以无精度损失地从d维降到d'维，当且仅当B矩阵的后d-d'个特征值全为0。所以此时截断到前d'个，并不会损失任何信息。

问题：
- 对B矩阵做特征值分解时来做近似，得到的近似具有什么性质？（比如，降维前后的距离矩阵的差的某个范数最小？）是否存在更优的近似策略？

自己实现的核心代码：

``` python
def MDS(points,top_k):
    # 计算距离矩阵D
    point_num,feature_num = points.shape
    assert top_k <= feature_num
    D = distance_matrix(points)
    
    # 计算内积矩阵B
    B = np.zeros((point_num,point_num))
    dist_ij = np.square(D)
    dist_io = np.sum(np.square(D),axis=1) / point_num
    dist_oj = np.sum(np.square(D),axis=0) / point_num
    dist_oo = np.sum(np.square(D)) / (point_num * point_num)
    for i in range(point_num):
        for j in range(point_num):
            B[i,j] = dist_ij[i,j] - dist_io[i] - dist_oj[j] + dist_oo
    B = - B / 2
    
    # 对B做特征值分解
    eigen_values, eigen_vectors = np.linalg.eigh(B)
    print(eigen_values)
  
    # 取前k个最大的特征值，及其对应的特征向量
    eigen_values = eigen_values[::-1][:top_k]
    eigen_vectors = eigen_vectors[:,::-1][:,:top_k]
    
    # 计算降维后的矩阵Z
    # 有些0特征值会产生数值误差，变成很小的负数，经过sqrt变成nan，引起后面计算错误。所以这里过一下abs。
    # 注意，B本身是半正定矩阵(B=Z^T*Z)，所以特征值本身应大于等于0，所以abs操作不会误伤原本的特征值。
    eigen_values = np.abs(eigen_values)
    Z = np.matmul(np.sqrt(np.diag(eigen_values)),eigen_vectors.T).T

    return Z
```

### 例子

#### 例1：

点的坐标：(2,0,0), (0,2,0),(0,0,2)

左图为原始数据（3维），右图为降维（2维）后的数据

经计算，任意两点的距离，在两图中均为2.82842712（即$2\sqrt{2}$）

![](MDS-1669986936051.jpeg)

#### 例2：

点的坐标：(2,0,0), (0,2,0), (0,0,2), (4,4,4)

有时MDS算法无法保证降维前后距离完全一致，只能取一个近似。本例中的四个点就是，无法在二维平面中找到距离与三维里完全相等的摆法。一个显而易见的例子是，在左图中，蓝、绿、橙三点之间距离相等，但降维后，右图里该三点距离不相等。

虽然无法精确保证距离，但还可以保留很多相对距离的信息，如左图中红点距离其余三点距离较远，在降维后依然能够体现出来。


![](MDS-1669987231447.jpeg)

#### 例3：

点的坐标：(2,1,4), (1,2,5), (1,1,6), (2,0,5),     (1,1,-4), (2,1,-6), (2,1,-5), (3,0,-4)

本例中，原先相近的点，降维后仍然相近；原先相远的点，降维后仍然相远。

![](MDS-1669987727564.jpeg)