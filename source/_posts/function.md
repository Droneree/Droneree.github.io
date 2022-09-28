---
title: 集合论与函数 
date: 2020-11-05 11:09:00
categories:
- 数理逻辑
tags: 
- 数学
- 集合论
mathjax: true
---



拉丁文（functio），含义为the action of performance。函数最重要的性质是**决定性**：同一输入只对应同一输出。<!--more-->

### 一、函数的定义

函数的现代定义（19世纪末，Dirichlet）：在集合论基础上，将函数视为关系的特例，特别之处就是决定性。设F为二元关系，F为函数是指
$$
(\forall x,y,z)(xFy\wedge xFz\to y=z)
$$
F为函数等价于
$$
x \in dom(F)\to \exists y!, \ F(x)=y
$$
空关系也是函数。

#### 函数的外延原则

设F，G为函数，则
$$
F=G \leftrightarrow [Dom(F)=Dom(G)\wedge (\forall x \in Dom(F))(F(x)=G(x))]
$$

#### 函数的集合

* ##### 定义域、值域、陪域

  设A，B为集合，F为从A到B的函数（记为$F:A\to B$），且$Dom(F)=A$，$Ran(F)\subseteq B$。则称A为函数F的定义域，$Ran(F)$为函数F的值域，B为函数F的陪域（codomain）。
  
* ##### 函数的集合

  记$B^A$为A到B的所有函数的集合，即$\{F|F:A\to B\}$，读作”B上A“。

* ##### 满射、单射与双射

  满射（onto）：值域等于陪域，$Ran(F)=B$

  单射（1-1）：$(\forall x,y \in A)(f(x)=f(y)\to x=y)$

  双射：(1-1 correspondence）：满射且单射。

### 二、函数的性质

### 三、函数的复合

* #### 函数的复合

  设F和G是函数，则$F\circ G$也是函数，且满足
  $$
  (1) \ Dom(F \circ G)=\{x|x \in Dom(G) \wedge G(x)\in Dom(F)\}\\
  (2)\ \forall x \in Dom(F\circ G),有F\circ G(X)=F(G(x))\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ 
  $$
  
* #### 复合函数的性质

  

### 四、反函数

#### 函数的逆关系

函数的逆关系不一定是函数，可能只是普通的二元关系。

对于单射函数$f:A\to B$，逆关系$f^{-1}$是函数，是$Ran(f)\to A$的双射函数，但不一定是$B\to A$的双射函数；

对于满射函数$f:A\to B$，逆关系$f^{-1}$不是函数；

对于双射函数$f:A\to B$，逆关系$f^{-1}$是$B\to A$的双射函数。

#### 反函数

对于$f$是$A\to B$的双射函数的情况，我们称逆关系$f^{-1}$是$f$的反函数。



<br/>

<br/>

<br/>

<small>*参考*</small>

<small>*主要整理自吴楠老师《离散数学》课堂讲授*</small>