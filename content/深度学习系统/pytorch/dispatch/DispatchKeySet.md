---
linktitle: DispatchKeySet
summary: Summary of DispatchKeySet
type: book
---
# DispatchKeySet
关于enum的小demo。从指定的项开始，递增；各项数字可以重复。
```c++
#include <iostream>
enum DAY {
      A=1, B, C, D=2, E, F, G
};
int main () {
   printf("A=%d\n",A);
   printf("B=%d\n",B);
   printf("C=%d\n",C);
   printf("D=%d\n",D);
   printf("E=%d\n",E);
   printf("F=%d\n",F);
   printf("G=%d\n",G);
   return 0;
}
/*
A=1
B=2
C=3
D=2
E=3
F=4
G=5
*/
```

DispatchKey从小到大如下：
```c++
Undefined = 0
CatchAll = Undefined
// ~~~~~~~~~~~~~ Functionality Keys ~~~~~~~~~~~~~ //
Dense
FPGA
....
AutocastXPU
AutocastCUDA
 // ~~~~~~~~~~~~~ WRAPPERS ~~~~~~~~~~~~~ //
FuncTorchBatched
FuncTorchVmapMode
...
TESTING_ONLY_GenericMode
PythonDispatcher
// ~~~~~~~~~~~~~ FIN ~~~~~~~~~~~~~ //
EndOfFunctionalityKeys
// ~~~~~~~~~~~~~ "Dense" Per-Backend Dispatch keys ~~~~~~~~~~~~~ //
StartOfDenseBackends
CPU
CUDA
...
PrivateUse2
PrivateUse3
EndOfDenseBackends = PrivateUse3

StartOfQuantizedBackends
QuantizedCPU
QuantizedCUDA
...
QuantizedPrivateUse2
QuantizedPrivateUse3
EndOfQuantizedBackends = QuantizedPrivateUse3

StartOfSparseBackends
SparseCPU
SparseCUDA
...
SparsePrivateUse2
SparsePrivateUse3
EndOfSparseBackends = SparsePrivateUse3

StartOfNestedTensorBackends
NestedTensorCPU
NestedTensorCUDA
...
NestedTensorPrivateUse2
NestedTensorPrivateUse3
EndOfNestedTensorBackends = NestedTensorPrivateUse3

StartOfAutogradFunctionalityBackends
AutogradCPU
AutogradCUDA
...
AutogradPrivateUse2
AutogradPrivateUse3
EndOfAutogradFunctionalityBackends = AutogradPrivateUse3

EndOfRuntimeBackendKeys = EndOfAutogradFunctionalityBackends

// ~~~~~~~~~~~~~~~~~ Alias Dispatch Keys ~~~~~~~~~~~~~~~~~~~~~ //
Autograd
CompositeImplicitAutograd
...
CompositeExplicitAutogradNonFunctional
StartOfAliasKeys = Autograd
EndOfAliasKeys = CompositeExplicitAutogradNonFunctional

// ~~~~~~~~~~~~~~~~~~~~ BC ALIASES ~~~~~~~~~~~~~~~~~~~~~~~~~~~ //
CPUTensorId = CPU
CUDATensorId = CUDA
...
PrivateUse3_PreAutograd = AutogradPrivateUse3
Autocast = AutocastCUDA
```

Dispatch Key Set 由BackendComponent(8bit)和DispatchKey(16bit)两部分组成

### 1.DispatchKeySet的各种构造函数
- DispatchKeySet()
	- repr\_全零
- DispatchKeySet(Full)
	- repr\_全1
- DispatchKeySet(FullAfter, DispatchKey t)
	- repr\_在某个位置之后全1
- DispatchKeySet(DispatchKey k)(191)
	- 将DispatchKey拆为functionality和backend两部分，在repr_的对应的这两个位置分别置1。
- DispatchKeySet(BackendComponent k)(183)
	- 将repr\_的backend对应的位置置1。
- DispatchKeySet(std::initializer_list\<DispatchKey\> ks)()
	- 多个DispatchKey一起设置（或起来）
- DispatchKeySet(std::initializer_list\<BackendComponent\> ks)()
	- 多个BackendComponent一起设置（或起来）

### 2.DispatchKeySet上的一些操作
- bool has(DispatchKey t)，bool has_backend(BackendComponent t)
	- repr\_上是否有某个DispatchKey或BackendComponent
- 
### 1.1 DispatchKeySet(DispatchKey k)
定义在c10/core/DispatchKeySet.h:191
- 若k是"functionality-only" keys，即k落在EndOfFunctionalityKeys之前，则直接对应位置1；
- 若k是"runtime" keys，即k落在EndOfFunctionalityKeys和EndOfRuntimeBackendKeys之间，也就是“笛卡尔积”的那部分，则需要把k拆成functionality\_k和backend\_k两部分，其中functionality\_k落在FunctionalityKeys内，backend\_k落在BackendComponent内。
- 否则，k置0。

问题："functionality-only" keys和"runtime" keys的区别是什么？
