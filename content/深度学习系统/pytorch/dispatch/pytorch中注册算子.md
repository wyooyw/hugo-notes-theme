---
linktitle: pytorch中注册算子
summary: Summary of pytorch中注册算子
type: book
---
# PyTorch中注册算子
两种方法：
- 内部算子
- 外部自定义算子
## 1.外部自定义算子
打断点b c10::RegisterOperators::checkSchemaAndRegisterOp_，然后bt，可以找到调用RegisterOperators().op来进行算子注册的代码：
```c++
//torch/csrc/jit/runtime/register_prim_ops_fulljit.cpp:850
static auto reg4 =
    torch::RegisterOperators()
        .op("_test::leaky_relu(Tensor self, float v=0.01) -> Tensor",
            &leaky_relu)
        .op("_test::cat(Tensor[] inputs) -> Tensor", &cat)
        .op("_test::get_first", &get_first);
```
实际发现，除了这里和一些测试代码，其他地方并没有使用RegisterOperators注册的算子。
阅读官方文档
https://github.com/pytorch/pytorch/blob/master/aten/src/ATen/core/op_registration/README.md
可以得知，RegisterOperators().op主要是用来注册外部的自定义算子。

## 2内部注册算子
```c++
//aten/src/ATen/BatchingRegistrations.cpp:1067
TORCH_LIBRARY_IMPL(aten, Batched, m) {
  // NB: Ideally we would like some operators, like size.int, to "fallthrough"
  // to the underlying implementation. However, because a BatchedTensor is a
  // Tensor wrapper, it only has one dispatch key (Batched) on it. The resolution
  // here is to just directly call the underlying implementation.
  m.impl("size.int", static_cast<int64_t (*)(const Tensor&, int64_t)>(native::size));
  m.impl("_add_batch_dim", native::_add_batch_dim);
  m.impl("_remove_batch_dim", native::_remove_batch_dim);
  m.impl("_make_dual", _make_dual_batching_rule);
  m.impl("_has_same_storage_numel", _has_same_storage_numel_batching_rule);
  m.impl("is_same_size", native::is_same_size);
  m.impl("_new_zeros_with_same_feature_meta", _new_zeros_with_same_feature_meta_batching_rule);
```