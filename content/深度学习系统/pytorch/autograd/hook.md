---
linktitle: hook
summary: Summary of hook
type: book
---
# Hook

python代码
```python
import torch

from torch.autograd import Variable

def backward_hook(grad):
    return grad
print(torch.__file__)
a = Variable(torch.randn((4,4)), requires_grad=True)
b = Variable(torch.randn((4,4)), requires_grad=True)
c = torch.matmul(a,b)
c.register_hook(backward_hook)
d = c.sum()
d.backward()
```
打断点：b torch::autograd::call_function
停留在torch/csrc/autograd/engine.cpp:808
按c跳过第一次断点，进入第二次断点
按n执行完auto fn后，执行p fn.pre_hooks().size()，输出1，也就是我们加的两个backwardhook。

此时执行bt，查看调用栈：
- THPEngine\_run\_backward (torch/csrc/autograd/python_engine.cpp:168)
- PythonEngine::execute ()