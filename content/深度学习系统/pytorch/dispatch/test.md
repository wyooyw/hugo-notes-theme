---
linktitle: test
summary: Summary of test
type: book
---
pytorch_debug/build/aten/src/ATen/core/TensorBody.h:2841
```c++
inline at::Tensor Tensor::mm(const at::Tensor & mat2) const {
    return at::_ops::mm::call(const_cast<Tensor&>(*this), mat2);
}
```

pytorch_debug/build/aten/src/ATen/Operators_3.cpp:3699
```c++
at::Tensor mm::call(const at::Tensor & self, const at::Tensor & mat2) {
    static auto op = create_mm_typed_handle(); // TypedOperatorHandler
    return op.call(self, mat2);
}
```
/home/yiou/opensource/pytorch_debug/build/aten/src/ATen/Operators_3.cpp:3692
```c++
static C10_NOINLINE c10::TypedOperatorHandle<mm::schema> create_mm_typed_handle() {
  return c10::Dispatcher::singleton() // Get dispatch table?
      .findSchemaOrThrow(mm::name, mm::overload_name) // find "mm" in dispatch table?
      .typed<mm::schema>();
}
```

## 2
接下来进入findSchemaOrThrow函数：
pytorch_debug/aten/src/ATen/core/dispatch/Dispatcher.cpp:79
```c++
OperatorHandle Dispatcher::findSchemaOrThrow(const char* name, const char* overload_name) {
  auto it = findSchema({name, overload_name});
  if (!it.has_value()) {
    // Check if we have ANYTHING; if that's the case, that means you're
    // missing schema
    auto it2 = findOp({name, overload_name});
    if (!it2.has_value()) {
      TORCH_CHECK(false, "Could not find schema for ", name, ".", overload_name);
    } else {
      TORCH_CHECK(false, "Could not find schema for ", name, ".", overload_name,
        " but we found an implementation; did you forget to def() the operator?");
    }
  }
  return it.value(); //返回值是一个OperatorHandler对象
}
```
从这里的逻辑来看，schema是比operator更高的一个概念。findSchema和findOp分别是找schema和operator。

这里看一下ezyang的博客里对schema的描述：
> To take advantage of all of the code generation which PyTorch brings, you need to write a *schema* for your operator. The schema gives a mypy-esque type of your function, and also controls whether or not we generate bindings for methods or functions on Tensor. You also tell the schema what implementations of your operator should be called for given device-layout combinations. Check out the [README in native](https://github.com/pytorch/pytorch/blob/master/aten/src/ATen/native/README.md) is for more information about this format.

感觉schema应该指的是native_functions.yaml里对函数的描述。

回到代码。看一下findSchema方法：
```c++
//pytorch_debug/aten/src/ATen/core/dispatch/Dispatcher.cpp:66
c10::optional<OperatorHandle> Dispatcher::findSchema(const OperatorName& overload_name) {
  auto it = findOp(overload_name);
  if (it.has_value()) {
    if (it->hasSchema()) {
      return it;
    } else {
      return c10::nullopt;
    }
  } else {
    return it;
  }
}
```
findOp方法：
```c++
//pytorch_debug/aten/src/ATen/core/dispatch/Dispatcher.cpp:56
c10::optional<OperatorHandle> Dispatcher::findOp(const OperatorName& overload_name) {
  return operatorLookupTable_.read([&] (const ska::flat_hash_map<OperatorName, OperatorHandle>& operatorLookupTable) -> c10::optional<OperatorHandle> {
    auto found = operatorLookupTable.find(overload_name);
    if (found == operatorLookupTable.end()) {
      return c10::nullopt;
    }
    return found->second;
  });
}
```
从代码推断，operatorLookupTable\_里存了各个算子的名称和其实现。OperatorName为算子的名称，OperatorHandle里应该有指向具体实现的指针？
“operatorLookupTable_”定义在
```c++
//pytorch_debug/aten/src/ATen/core/dispatch/Dispatcher.h:292
LeftRight<ska::flat_hash_map<OperatorName, OperatorHandle>> operatorLookupTable_;
```
LeftRight是一种特殊的数据结构，定义在
```
// c10/util/LeftRight.h
// LeftRight wait-free readers synchronization primitive
// https://hal.archives-ouvertes.fr/hal-01207881/document
//
// LeftRight is quite easy to use (it can make an arbitrary
// data structure permit wait-free reads), but it has some
// particular performance characteristics you should be aware
// of if you're deciding to use it:
//
//  - Reads still incur an atomic write (this is how LeftRight
//    keeps track of how long it needs to keep around the old
//    data structure)
//
//  - Writes get executed twice, to keep both the left and right
//    versions up to date.  So if your write is expensive or
//    nondeterministic, this is also an inappropriate structure
```
关于LeftRight这里不再深入探究。
## 3.
OperatorHandler对象被返回后，执行了该对象的typed方法，如下：
```c++
//pytorch_debug/aten/src/ATen/core/dispatch/Dispatcher.h:372
template<class FuncType>
  TypedOperatorHandle<FuncType> typed() const {
    // NB: This assert is not 100% sound: you can retrieve a typed() operator
    // handle prior to ANY C++ signature being registered on the operator
    // and the check will say everything is OK (at which point you can then
    // smuggle in a kernel that is typed incorrectly).  For everything
    // in core library this won't happen, because all the static registrations
    // will be done by the time a typed() handle is acquired.
#if !defined C10_MOBILE
    operatorDef_->op.assertSignatureIsCorrect<FuncType>();
#endif
    return TypedOperatorHandle<FuncType>(operatorIterator_);
  }
```
其中operatorDef\_的类型为**OperatorDef**，operatorDef_->op的类型为**OperatorEntry**；operatorIterator是list\<OperatorDef\>类型。
TypedOperatorHandle是OperatorHandle类的子类。operatorDef\_和operatorIterator\_是OperatorHandle类的两个属性
```c++
//Dispatcher.h:310
// Storing a direct pointer to the OperatorDef even though we
  // already have the iterator saves an instruction in the critical
  // dispatch path. The iterator is effectively a
  // pointer-to-std::list-node, and (at least in libstdc++'s
  // implementation) the element is at an offset 16 bytes from that,
  // because the prev/next pointers come first in the list node
  // struct. So, an add instruction would be necessary to convert from the
  // iterator to an OperatorDef*.
  Dispatcher::OperatorDef* operatorDef_;
  // We need to store this iterator in order to make
  // Dispatcher::cleanup() fast -- it runs a lot on program
  // termination (and presuambly library unloading).
  std::list<Dispatcher::OperatorDef>::iterator operatorIterator_;
```
后续还要看一下这几个类型之间的关系。