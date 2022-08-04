---
linktitle: Linux常用命令
summary: Summary of Linux常用命令
type: book
---
# Linux常用命令
## 1.tar
最简单的打包/解包命令：
```shell
# 打包为.tar文件
tar -cvf [文件名].tar [文件目录]
# 解压到当前目录
tar -xvf [文件名].tar
```

实际上tar命令的功能不止这些。
tar命令用来打包一个目录，它支持三种格式：".tar"、".bz2"和".gz"。tar只打包不压缩，bz2和gz会进行压缩。
下面详细说明。下文摘自[该博客](https://blog.csdn.net/weixin_39270987/article/details/122958566)的总结
### 1.1 打包
```shell
tar -cvf [文件名].tar [文件目录] //打包成.tar文件 
tar -jcvf [文件名].tar.bz2 [文件目录] //打包成.bz2文件 
tar -zcvf [文件名].tar.gz [文件目录] //打包成.gz文件
```

### 1.2 解压
```shell
tar -xvf [文件名].tar //解压到当前文件
tar -xvf [文件名].tar -C [文件目录] //将.tar文件解压到指定目录
tar -jxvf [文件名].tar.bz2 -C [文件目录] //解压.bz2文件到指定目录
tar -zxvf [文件名].tar.gz -C [文件目录] //解压.gz文件到指定目录
```
### 1.3 常用选项
-c 建立新的压缩文件  
-C 指定解压目录，该目录必须存在  
-x 从压缩的文件中提取文件  
-j 支持bzip2解压文件  
-f 指定压缩文件  
-v 显示操作过程  
-z 支持gzip解压文件