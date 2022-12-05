---
linktitle: ZFC公理
summary: Summary of ZFC公理
type: book
---
# ZFC公理
## 1.ZFC公理

**外延公理**：判断两集合相等

**空集公理**：保证空集存在

**对偶公理、并集公理、幂集公理、子集公理**：从已有集合构造新集合

**无穷公理**：构造自然数序列，“引出现代数学无限奥妙”

**替换公理**：

**正规公理**：规定了什么不能是集合

**选择公理**：

### 1.1 外延公理
$$
\forall x y(\forall z(z \in x \leftrightarrow z \in y) \leftrightarrow x=y)
$$
该定理说明，**两个集合相等**，是指**两个集合包含的元素相等**。

集合由其包含的元素所确定。

### 1.2 空集公理
$$
\exists x \forall y \neg(y \in x)
$$
满足条件的$x$记作$\emptyset$，称为空集。

定理保证**空集存在**。集合论的任意模型都包含空集。

### 1.3 对偶公理
$$
\forall x y \exists u \forall z(z \in u \leftrightarrow z=x \vee z=y)
$$
合并到一起

举例：{a,b},{c,d} -> {{a,b},{c,d}}

对偶公理和并集公理保证了有限集合的存在性

### 1.4 并集公理
$$
\forall x \exists u \forall y(y \in u \leftrightarrow(\exists z(z \in x \wedge y \in z))) .
$$
直观：打破屏障

举例：{{a,b},{c,d}} -> {a,b,c,d}

**1）并集符号$\cup$**

记 $u=\cup x$

平时写的$u=a \cup b$，其实是$u=\cup \lbrace a,b \rbrace$的简写。

**2）交集$\cap$**

设满足下式的$v$：
$$
\forall z(z \in v \leftrightarrow \forall y(y \in x \rightarrow z \in y))
$$
记为$v= \cap x$。

同样若$x=\lbrace x_1,x_2,...,x_n \rbrace$，则$v=\cap x=\cap \lbrace x_1,x_2,...,x_n \rbrace$也可写为$v=x_{1}\cap x_2...\cap x_n$

如何保证交集存在？

**3）关于空集**

- $\cup \emptyset= \emptyset$    
因为$x$为空集，所以
![](ZFC公理-1665150588171.jpeg)
等价式右侧、合取前面的"$z \in x$"一定为假，所以等价式左侧的“$y \in u$”只能为假，所以$u$只能是空集。

- $\cup \lbrace \emptyset \rbrace = \emptyset$   
等价式右侧，z只能是空集，所以合取后面的"$y \in x$"一定为假，所以等价式左侧的“$y \in u$”只能为假，所以$u$只能是空集。

- $\cap \emptyset$ 不存在
假设存在，设$\mu=\cap \emptyset$，则有

$$

\forall z(z \in \mu \leftrightarrow \forall y(y \in \emptyset \rightarrow z \in y)) \\
$$
由于蕴含式左侧“$y \in \emptyset$”为假，所以整个蕴含式为真，所以等价符号左侧“$z \in \mu$”为真，即$\mu$是全集。而**全集是不存在的**(因为假设全集存在,会与正则公理矛盾)，所以假设错误。

### 1.5 幂集公理
$$
\forall x \exists y(\forall z(z \in y \leftrightarrow \forall u(u \in z \rightarrow u \in x)))
$$
直观:集合存在幂集

### 1.6 子集公理
$$
\forall x_1 \cdots x_n \forall x \exists y \forall z(z \in y \rightarrow z \in x \wedge \phi) .
$$
其中$\phi$是集合论语言的公式，仅出现自由变元$x_1,...,x_n,x,z$,不出现$y$。

直观：**集合x存在子集y**。其中y中元素是根据条件$\phi$筛选出来的，即$y=\lbrace z \in x | \phi \rbrace$

**1) 与朴素集合论的区别**

朴素集合论中，集合是“满足某一条件的对象的全体”，所以$S=\lbrace x|P(x) \rbrace$就是集合。

但在ZFC公理中，仅$S=\lbrace x|P(x) \rbrace$不能说明S是集合，必须还说明**S是某一已知集合的子集**才行。

考虑罗素悖论：$A=\lbrace x|x \notin x \rbrace$，在ZFC公理中不能说A是集合，因为没有说A是某一已知集合的子集。(在后续”正则公理“中，则是直接禁止了该类集合的存在)

**2) 公理模式**

子集公理是一种"**公理模式**"。它是无限多条公理的统一形式。(因为其$\phi$并没有明确指定)

后续的"置换公里"也是一种"公理模式"

### 1.7 无穷公理
$$
\exists x\left(\emptyset \in x \wedge\left(\forall y\left(y \in x \rightarrow y^{+} \in x\right)\right)\right)
$$
其中$y^+$表示集合$y \cup \lbrace y \rbrace$

这里存在的$x$称为归纳集,无穷公理保证归纳集是存在的

直观:自然数集$N=\lbrace 0,1,2,.... \rbrace$是归纳集,是存在的

无穷公理如何保证集合里其他元素是合法的?

### 1.8  替换公理

假设$\phi$是集合论语言的公式， 仅出现自由变元$x_1,...,x_n,u,v$ ,不出现变元$y$ , 则

$$
\forall x_1 \cdots x_n \forall x(\psi \rightarrow \exists y \forall v(v \in y \leftrightarrow \exists u \in x \phi(u, v)))
$$

其中$\psi$是如下公式:

$$
\forall u \in x \forall v_1 v_2\left(\phi\left(u, v_1\right) \wedge \phi\left(u, v_2\right) \rightarrow v_1=v_2\right)
$$

直观:
- $\phi (u,v)$是一种"函数",将$u$映射到$v$,一个$u$不能对应两个$v$.
- 替换公里表明,任何集合在一个函数下的象仍是一个集合.

### 1.9 正则公理
$$
\forall x(x \neq \emptyset \rightarrow \exists y(y \in x \wedge y \cap x=\emptyset))
$$
直观:规定$\lbrace x|x \in x \rbrace$不是集合

### 1.10 选择公理
