---
linktitle: permute
summary: Summary of permute
type: book
---
断点打在at::native::permute，停在aten/src/ATen/native/TensorShape.cpp:1165
```c++
Tensor permute(const Tensor& self, IntArrayRef dims) {
  DimVector new_sizes, new_strides;
  std::vector<int64_t> _;
  std::tie(new_sizes, new_strides, _) = _permute_size_stride_estimation(self, dims);
  return self.as_strided(new_sizes, new_strides);
}
```
其中\_permute\_size\_stride\_estimation函数（aten/src/ATen/native/TensorShape.cpp:1134），计算了新的shape和stride。基本原理就是，新的shape和stride是原shape和stride的置换，且其置换顺序与permute中定义的顺序相同。关键代码如下：
```c++
for (const auto i : c10::irange(ndim)) {
    const auto d = maybe_wrap_dim(dims[i], ndim);
    TORCH_CHECK(!seen_dims[d],
        "permute(): duplicate dims are not allowed.");
    seen_dims[d] = true;
    wrapped_dims[i] = d;
    new_sizes[i] = old_sizes[d];
    if (is_strided_layout) {
      new_strides[i] = old_strides[d];
    }
  }
  return std::make_tuple(new_sizes, new_strides, wrapped_dims);
```
self.as_strided则是将new\_sizes和new\_strides赋值给self对象？