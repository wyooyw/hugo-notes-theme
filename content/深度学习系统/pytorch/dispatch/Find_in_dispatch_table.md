---
linktitle: Find_in_dispatch_table
summary: Summary of Find_in_dispatch_table
type: book
---
打断点b c10::impl::OperatorEntry::lookup，进入
OperatorEntry类的loop方法（aten/src/ATen/core/dispatch/OperatorEntry.h:175）
```c++
const KernelFunction& lookup(DispatchKeySet ks) const {
    const auto idx = ks.getDispatchTableIndexForDispatchKeySet();
    if (C10_UNLIKELY(idx == -1)) {
      reportError(ks.highestPriorityTypeId());
    }
    const auto& kernel = dispatchTable_[idx];
    // A valid kernel *always* has a boxed kernel and *may* have an
    // unboxed kernel. However, we typically do unboxed calls in at::
    // APIs, where the kernel 1) will very likely be valid and 2)
    // should have an unboxed kernel. Checking the unboxed kernel
    // first will allow us to avoid touching the boxed kernel at all
    // in the common case.
    if (C10_UNLIKELY(!kernel.isValidUnboxed())) {
      if (!kernel.isValid()) {
        reportError(ks.highestPriorityTypeId());
      }
    }
    return kernel;
  }
```
lookup函数干的事情大致是，根据dispatch key set，拿到需要的kernel函数。
关注一下getDispatchTableIndexForDispatchKeySet方法和dispatchTable\_对象。
一个OperatorEntry对应一个算子，其内部记录了该算子在不同dispatch key下的不同实现。
OperatorEntry里核心的两个内容式dispatchTable和kernels，
### 2）dispatchTable_
dispatchTable\_的定义如下：
```c++
std::array<KernelFunction, c10::num_runtime_entries> dispatchTable_;
```
dispatchTable\_是OperatorEntry类的成员属性，是一个**数组**，里面存了**一个算子的各种实现的KernelFunction**。选用哪个KernelFunction**取决于DispatchKeySet**，通过其getDispatchTableIndexForDispatchKeySet()方法可以得到实际需要的KernelFunction在dispatchTable\_的下标。
接下来关注一下OperatorEntry.cpp中对dispatchTable\_的各种操作。

#### 修改操作
对dispatchTable\_唯一修改的地方，是在

```c++
//aten/src/ATen/core/dispatch/OperatorEntry.cpp:373
void OperatorEntry::updateDispatchTableEntry_(const c10::Dispatcher& dispatcher, DispatchKey dispatch_key) {
  const auto dispatch_ix = getDispatchTableIndexForDispatchKey(dispatch_key);
  if (C10_UNLIKELY(dispatch_ix == -1)) {
    return;
  }
  dispatchTable_[dispatch_ix] = computeDispatchTableEntry(dispatcher, dispatch_key);
  dispatchKeyExtractor_.setOperatorHasFallthroughForKey(dispatch_key, dispatchTable_[dispatch_ix].isFallthrough());
}
```