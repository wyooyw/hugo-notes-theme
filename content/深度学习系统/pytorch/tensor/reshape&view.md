---
linktitle: reshape&view
summary: Summary of reshape&view
type: book
---
## 0.相关资源
一篇讲Tensor、view与reshape的比较详细的博客：
https://blog.csdn.net/Flag_ing/article/details/109129752

python代码：
```python
import torch
a = torch.randn((2,3,4))
b = a.reshape(2,12)
print(b)
```
gdb添加断点：
```
b Tensor::reshape
```
程序执行停在：
```c++
//pytorch_debug/build/aten/src/ATen/core/TensorBody.h:3066
inline at::Tensor Tensor::reshape(at::IntArrayRef shape) const {
    return at::_ops::reshape::call(const_cast<Tensor&>(*this), shape);
}
```
call函数：
```
// aten::reshape(Tensor(a) self, int[] shape) -> Tensor(a)
at::Tensor reshape::call(const at::Tensor & self, at::IntArrayRef shape) {
    static auto op = create_reshape_typed_handle();
    return op.call(self, shape);
}
```
其中create_reshape_typed_handle会从dispatch table里拿到reshape操作对应的OperatorHandle对象，包装为TypedOperatorHandle对象后返回（也就是这里的op）。
但是怎么找到op计算时具体的函数呢？

尝试断点打在at::native::reshape，执行，停在这里
```c++
//pytorch_debug/aten/src/ATen/native/TensorShape.cpp:1321
Tensor reshape(const Tensor& self, IntArrayRef proposed_shape) {
  if (self.is_sparse()) {
    AT_ERROR("reshape is not implemented for sparse tensors");
  }
  DimVector shape = infer_size_dv(proposed_shape, self.numel());

  if (self.is_mkldnn()) {
    return at::_mkldnn_reshape(self, shape);
  }

  // `computeStride` returns the proper strides to use if this
  // `reshape` can be just a view.
  auto stride = at::detail::computeStride(self.sizes(), self.strides(), shape);

  // NB: Even though we have viewable geometry and the target strides here,
  //     we do not just call `as_strided` on `self` because the backward

  //     for `as_strided` is not as efficient as that of `view` (since the
  //     former is meant to handle general cases).
  //
  //     Similarly we don't call `view` because it duplicates some of the work
  //     we've already done, and instead call our internal/private operator
  //     `_reshape_alias` that essentially does the same thing as `view` and
  //     `as_strided` without any of the extra overhead.
  if (stride.has_value()) {
    // Temporary check to revert to the old behavior/view in cases where the
    // device is not supported (e.g. for XLA the operation is not supported
    // so we use `view` instead).
    //
    // We need to do the checks here instead of in `native_functions.yaml`
    // to preserve backwards compatibility.
    if (!self.is_xla() && !self.is_lazy() && !self.is_ipu()) {
      return self._reshape_alias(shape, stride.value());
    } else {
      return self.view(shape);
    }
  }
  return at::_unsafe_view(self.clone(at::MemoryFormat::Contiguous), shape);
}
```
大致逻辑可以看出来，**如果self是连续的**，则调用view，但这里由于view会做一些重复的事情，所以其实会**直接调用_reshape_alias**，猜测view里是先做一些初始工作，然后也调用的_reshape_alias。如果self是**不连续的**，则先将self**拷贝**，再在拷贝后的Tensor上执行**\_unsafe\_view**。
下面仔细看一下infer_size_dv，computeStride，\_reshape\_alias，view和_unsafe_view五个函数。

### 1）infer_size_dv
打断点b at::infer_size_dv，进入该函数：
```c++
//pytorch_debug/aten/src/ATen/InferSize.h:72
inline at::DimVector infer_size_dv(IntArrayRef shape, int64_t numel) {
  auto res = at::DimVector(shape);
  infer_size_impl(shape, numel, res);
  return res;
}
```
首先，at::DimVector将shape包装为一个DimVector对象（DimVector实际是一个SmallVector，调用的构造函数在pytorch_debug/c10/util/SmallVector.h:1324）。
然后，**infer\_size\_impl函数负责检查shape的合法性，且当shape中包含一个“-1”时，推断出其实际值**。该函数位置在pytorch_debug/aten/src/ATen/InferSize.h:20


### 2）computeStride
打断点b at::detail::computeStride，进入computeStride函数，位置为pytorch_debug/aten/src/ATen/TensorUtils.cpp:400。其调用了**computeStride_impl**函数，位置为
pytorch_debug/aten/src/ATen/TensorUtils.cpp:314。

```c++
// On a high level,
// 1. separate `oldshape` into chunks of dimensions, where the dimensions are
//    ``contiguous'' in each chunk, i.e., oldstride[i] = oldshape[i+1] *
//     oldstride[i+1]

// 2. `newshape` must be able to be separated into same number of chunks as
//    `oldshape` was separated into, where each chunk of newshape has matching
//    ``numel'', i.e., number of subspaces, as the corresponding chunk of
//    `oldshape`.
//
// templatized for DimVector and IntArrayRef use cases,
// see overloads of computeStride() below.
//
template <typename ResultVec, typename NewShapeVec, typename Numel>
inline c10::optional<ResultVec> computeStride_impl(
    const NewShapeVec& oldshape,
    const NewShapeVec& oldstride,
    const NewShapeVec& newshape,
    ResultVec toResult(const NewShapeVec&)
) {
```

首先从该函数的注释可以看出，将tensor的**轴划分成多个chunk**，划分依据是每个**chunk内部是连续的**。若newshape相对oldshape的改变，是在chunk内，则只需要创建新view，无需申请新data内存；若newshape相对oldshape的改变**跨越了chunk的边界，则需要申请新data内存**。关键实现代码如下：

```c++
  int64_t view_d = (int64_t)newshape.size() - 1;
  // stride for each subspace in the chunk
  Numel chunk_base_stride = oldstride.back();
  // numel in current chunk
  Numel tensor_numel = 1;
  Numel view_numel = 1;
  for (int64_t tensor_d = oldshape.size() - 1; tensor_d >= 0; tensor_d--) {
    tensor_numel *= oldshape[tensor_d];
    // if end of tensor size chunk, check view
    if ((tensor_d == 0) ||
        (oldshape[tensor_d - 1] != 1 &&
        oldstride[tensor_d - 1] != tensor_numel * chunk_base_stride)) {
      while (view_d >= 0 &&
            (view_numel < tensor_numel || newshape[view_d] == 1)) {
        newstride[view_d] = view_numel * chunk_base_stride;
        view_numel *= newshape[view_d];
        view_d--;
      }
      if (view_numel != tensor_numel) {
        return c10::nullopt;
      }
      if (tensor_d > 0) {
        chunk_base_stride = oldstride[tensor_d - 1];
        tensor_numel = 1;
        view_numel = 1;
      }
    }
  }
  if (view_d != -1) {
    return c10::nullopt;
  }
  return newstride;
```
接下来做个实验来验证上面的结论，如下代码所示。可以看到虽然b不连续，但c仍然和b共享了一份数据，d因为其reshape时跨越了chunk的边界，所以没有和c共享数据，而是使用了新的内存。
```python
import torch
a = torch.randn((2,2,2,2))
b = a.permute(2,3,0,1)
c = b.reshape(4,4)
d = b.reshape(2,8)
print(f"addr={a.storage().data_ptr()}, stride={a.stride()}")
print(f"addr={b.storage().data_ptr()}, stride={b.stride()}")
print(f"addr={c.storage().data_ptr()}, stride={c.stride()}")
print(f"addr={d.storage().data_ptr()}, stride={d.stride()}")
'''
addr=94432814628928, stride=(8, 4, 2, 1)
addr=94432814628928, stride=(2, 1, 8, 4)
addr=94432814628928, stride=(1, 4)
addr=94432814636544, stride=(8, 1)
'''
```
将c和d带入到前面的代码中也可以得出，c可以顺利执行到函数最后一行，而d会则会执行到"return c10::nullopt"。

### 3）\_reshape\_alias
self.\_reshape\_alias会在查找dispatch table后，进入
\_reshape\_alias函数（pytorch\_debug/aten/src/ATen/native/TensorShape.cpp:1360）。根据该函数的注释可以得知，这个函数仅供reshape使用，为的是避免直接执行view时会重复执行infer\_size\_dv和computeStride。

```c++
Tensor _reshape_alias(const Tensor& self, IntArrayRef sizes, IntArrayRef strides) {
  // This is only used by `reshape` in cases where it would otherwise have dispatched
  // to `view`. This removes the overhead of calling `view` which duplicates some of
  // the work that's already been done (`infer_size_dv` and `computeStride`).
  return alias_with_sizes_and_strides(self, sizes, strides);
}
```

而后进入alias_with_sizes_and_strides函数（aten/src/ATen/native/TensorShape.cpp:1297），代码如下：
```c++
template <typename Vec>
Tensor alias_with_sizes_and_strides(
    const Tensor& self,
    const Vec& sizes,
    const Vec& strides) {
  //caller should make sure that sizes and strides are valid for self
  //(storage is sufficient, strides are non-negative, strides and sizes array size is the same)
  Tensor self_;
  if (self.is_quantized()) {
    self_ = at::detail::make_tensor<QTensorImpl>(
      c10::TensorImpl::VIEW, Storage(self.storage()), self.key_set(), self.dtype(), get_qtensorimpl(self)->quantizer());
    auto* self_tmp_ = self_.unsafeGetTensorImpl();
    self_tmp_->set_storage_offset(self.storage_offset());
    self_tmp_->set_sizes_and_strides(sizes, strides);
  } else {
    self_ = at::detail::make_tensor<TensorImpl>(
      c10::TensorImpl::VIEW, Storage(self.storage()), self.key_set(), self.dtype());
    auto* self_tmp_ = self_.unsafeGetTensorImpl();
    self_tmp_->set_storage_offset(self.storage_offset());
    self_tmp_->set_sizes_and_strides(sizes, strides);
  }
  namedinference::propagate_names(self_, self);
  return self_;
}
```
该函数做的事情很简单，就是新建了一个Tensor，**其storage指向原tensor的storage**，然后shape和stride设成新的。最后的propagate_names函数是一个实验阶段的功能，而且如果没有特意为张量设定names属性，则该函数不会做任何事情，可以忽略。

### 4）view
self.view经过dispatch后，进入at::native::view函数(aten/src/ATen/native/TensorShape.cpp:3274)，而后直接调用了view_impl函数(aten/src/ATen/native/TensorShape.cpp:2904)

```c++
inline Tensor view_impl(const Tensor& self, IntArrayRef size) {
  at::DimVector inferred_size = at::infer_size_dv(size, self.numel());
  auto stride = at::detail::computeStride(self.sizes(),
                                          self.strides(),
                                          inferred_size);
  TORCH_CHECK(stride.has_value(), "view size is "
    "not compatible with input tensor's size and stride (at least one dimension"
    " spans across two contiguous subspaces). Use .reshape(...) instead.");
  return alias_with_sizes_and_strides(self, inferred_size, *stride);
}
```

可以看出，view_impl干的事情很简单，先计算新的shape，再计算新的stride，**如果stride产生不连续问题（shape的变化超越了chunk的边界），则报错**（而不是像reshape那样会复制一个新的storage）。最后生成**与原Tensor共享storage的新Tensor**。其调用的infer_size_dv，computeStride和alias_with_sizes_and_strides函数在前文都已经分析过。

### 5）\_unsafe\_view
\_unsafe\_view函数(aten/src/ATen/native/TensorShape.cpp:2917)
和上面的at::native::view函数完全一样，都是调用了view_impl函数。
为何unsafe？代码里的解释：
```c++
// NOTE [ Unsafe View ]
// _unsafe_view() differs from view() in that the returned tensor isn't treated
// as a view for the purposes of automatic differentiation. (It's not listed in
// VIEW_FUNCTIONS in gen_inplace_or_view_type.py).  It's only safe to use if the `self` tensor
// is temporary. For example, the viewed tensor here (a + b) is discarded immediately
// after viewing:
//
//  res = at::_unsafe_view(a + b, size);
//
// This is a hack because in-place operations on tensors treated like views
// can be much more expensive than the same operations on non-view tensors.
inline Tensor view_impl(const Tensor& self, IntArrayRef size) {
```
只看懂了前半段，通过\_unsafe_\view计算的张量，不会被自动求导所记录，所以只能处理临时的张量。（可能在self.view到at::native::view之间，经历了自动求导层，会对该操作进行记录？）