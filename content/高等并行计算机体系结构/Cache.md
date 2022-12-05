---
linktitle: Cache
summary: Summary of Cache
type: book
---
关于cache的文章：https://zhuanlan.zhihu.com/p/455717838

### 阻塞缓存和非阻塞缓存

阻塞缓存：缓存系统在遇到cache miss的情况下将不能接受新的load/store miss

非阻塞缓存：可以接受多个cache miss的缓存系统，其使用了Miss Status Holding Registers (MSHRs)的结构来保存未解决的cache miss的信息。

