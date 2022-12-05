---
linktitle: PIM实验
summary: Summary of PIM实验
type: book
---
# PIM实验
全精度的策略：
torchquanter/utils/utils.py  func_bitreverse
量化的策略：
examples/weightReverse.py update_continuous_reverse
注意：
- 模型代码/量化代码 均需修改
- 只针对普通卷积和pointwise卷积做策略，不对depthwise卷积做策略。