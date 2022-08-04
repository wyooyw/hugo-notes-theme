---
linktitle: 传统IR模型
summary: Summary of 传统IR模型
type: book
---
# 传统IR模型
## 1.语言模型方法
![](传统IR模型-1659532911165.jpeg)
### 1.1 TF-IDF
![](传统IR模型-1659532935378.jpeg)
![](传统IR模型-1659532993349.jpeg)
log使得取值范围缩小
如果一个词在很多文章中都会出现，那它的IDF将会较小，如the、a、but等词。
![](传统IR模型-1659533235735.jpeg)
### 1.2 BM25
![](传统IR模型-1659533267048.jpeg)
![](传统IR模型-1659533490934.jpeg)

## 2.神经翻译语言模型
神经翻译语言模型（Neural Translation Language Model, NTLM）
Information retrieval as statistical translation,  SIGIR, 1999.
![](传统IR模型-1659533900826.jpeg)

## 3.泛化语言模型
泛化语言模型（Generlize Language Model, GLM）
Word Embedding based Generalized Language Model for Information Retrieval, SIGIR, 2015
![](传统IR模型-1659534161277.jpeg)

## 4.顺序依赖模型
顺序依赖模型（Sequential Dependence Model, SDM）
A Markov Random Field model for term dependencies, 
![](传统IR模型-1659534232136.jpeg)

## 总结
传统IR模型的**优势**：
高效处理大规模数据
无需注释标签

**劣势**：
词汇失配（vocabulary mismatch）
缺乏对查询与文档的深入理解（shallow understanding）
![](传统IR模型-1659534502387.jpeg)