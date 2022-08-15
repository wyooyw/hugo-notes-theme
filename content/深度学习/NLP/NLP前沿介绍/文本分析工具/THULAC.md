---
linktitle: THULAC
summary: Summary of THULAC
type: book
---
# THULAC
## 简介
实现中文的分词和词性标注

github地址：https://github.com/thunlp/THULAC-Python

## 快速上手

### 安装
#### 源码安装
见官网readme
#### pip安装
```shell
pip install thulac
```
### 示例代码
```python
thu1 = thulac.thulac()  #默认模式

text = thu1.cut("我爱北京天安门", text=True)  #进行一句话分词

print(text)

  

thu1 = thulac.thulac(seg_only=True)  #只进行分词，不进行词性标注

thu1.cut_f("input.txt", "output.txt") #注意，input.txt需要以gbk格式保存，output.txt也要以gbk格式查看。vscode右下角可以设置。
```

## 报错解决
```
AttributeError: module 'time' has no attribute 'clock'
```
参考https://github.com/thunlp/THULAC-Python/issues/113
python3.8之后time废除了clock方法。
解决方法：
1.将thulac/character/CBTaggingDecoder.py第170行改为
```python
start = time.perf_counter()
```
2.python降级

## bug
### 1.filt失效
开启seg_only后，filt失效了
使用fast_cut后，file也会失效

### 2.fast_cut和cut结果不同