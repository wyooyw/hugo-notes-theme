---
linktitle: TORCH_LIBRARY_IMPL
summary: Summary of TORCH_LIBRARY_IMPL
type: book
---
# TORCH_LIBRARY_IMPL
定义在torch/library.h:949
```c++
#define TORCH_LIBRARY_IMPL(ns, k, m) _TORCH_LIBRARY_IMPL(ns, k, m, C10_UID)
```

\_TORCH_\LIBRARY_\IMPL如下，可以看到里面涉及了DispatchKey名为k，说明TORCH_LIBRARY_IMPL的中间的参数k就是指dispatch key。
```c++
//torch/library.h:957
/// \private
///
/// The above macro requires an extra unique identifier (uid) to prevent
/// variable name collisions. This can happen if TORCH_LIBRARY_IMPL is called
/// multiple times with the same namespace and dispatch key in the same
/// translation unit.
#define _TORCH_LIBRARY_IMPL(ns, k, m, uid)                           \
  static void C10_CONCATENATE(                                       \
      TORCH_LIBRARY_IMPL_init_##ns##_##k##_, uid)(torch::Library&);  \
  static const torch::detail::TorchLibraryInit C10_CONCATENATE(      \
      TORCH_LIBRARY_IMPL_static_init_##ns##_##k##_, uid)(            \
      torch::Library::IMPL,                                          \
      c10::guts::if_constexpr<c10::impl::dispatch_key_allowlist_check( \
          c10::DispatchKey::k)>(                                     \
          []() {                                                     \
            return &C10_CONCATENATE(                                 \
                TORCH_LIBRARY_IMPL_init_##ns##_##k##_, uid);         \
          },                                                         \
          []() { return [](torch::Library&) -> void {}; }),          \
      #ns,                                                           \
      c10::make_optional(c10::DispatchKey::k),                       \
      __FILE__,                                                      \
      __LINE__);                                                     \
  void C10_CONCATENATE(                                              \
      TORCH_LIBRARY_IMPL_init_##ns##_##k##_, uid)(torch::Library & m)
```