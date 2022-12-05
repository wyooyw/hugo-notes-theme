---
linktitle: gdb调试pytorch
summary: Summary of gdb调试pytorch
type: book
---
# gdb调试pytorch
index.py里写一个cpu的torch.mul
```
b at::native::_matmul_impl
run
import index
```
然后就会停在
pytorch_debug/aten/src/ATen/native/LinearAlgebra.cpp:1747
### 关于gdb
栋哥的教程：
https://www.cnblogs.com/Jack47/p/survive-in-gdb.html
栋哥博客里推荐的陈浩的教程：
https://blog.csdn.net/haoel/article/details/2879