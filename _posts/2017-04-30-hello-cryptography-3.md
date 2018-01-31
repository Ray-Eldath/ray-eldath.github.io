---
layout: post
title: 你好，密码学(3)：离散概率（续）
date: 2017-04-30
categories: 技术
tags: 技术 你好，密码学 系列文章
cover: http://ray-eldath.image.alimmdn.com/header/hello-cryptography-3.png
---

# 导览

课程地址请见[Coursera](https://www.coursera.org/learn/crypto/lecture/JkDRg/discrete-probability-crash-course-cont)。

对于离散概率的更详细解释，请见：[WikiBooks (English)](https://zhuanlan.zhihu.com/p/High%20School%20Mathematics%20Extensions/Discrete%20Probability)

> 注意：文章中对专业名词进行解释的链接均来自[中文维基百科](https://zh.wikipedia.org/)，中国大陆读者若想访问请自备... ...
> 欢迎访问[作者博客](http://ray-eldath.tech/)。

**若未阅读上篇文章密码学I：离散概率的，*强烈*建议前去阅读。**

# 独立性

## 定义

**独立性**：若存在两事件$A$、$B$，$A$事件的发生无法向你提供关于$B$事件的**任何**信息，则称$A$、$B$两事件是互相独立的。即：

$$
Pr[A\;\:\mathtt{and}\;\:B]=Pr[A]\cdot Pr[B]
$$

> 通常来讲，独立性衡量的是两事件间互不相关的程度。

## 随机变量中的独立性

若有两个在集合$V$中取值的随机变量$X,Y$，那么如果这两个随机变量满足：

$$
\forall a,b\in V\\
Pr[X=a\;\:\mathtt{and}\;\:X=b]\;\;=\;\;Pr[X]\cdot Pr[Y=b]
$$

> 这意味着即使你知道$X=a$你也不知道关于$y$的取值的任何信息。

# 异或运算（XOR，$\oplus$）

这是一种在密码学中*相当重要和基础*的运算。

> 关于其重要的原因我们将稍后讲到。

## 定义

一种对数值$A$和$B$进行的运算，记为：

$$
A\oplus B
$$

其运算基本规则为：**两两数值相同时为假，而数值不同时为真**。

>
> 请见：[维基百科 - 逻辑异或（中文 - 大陆简体）](https://zh.wikipedia.org/zh-cn/%25E9%2580%25BB%25E8%25BE%2591%25E5%25BC%2582%25E6%2588%2596) 

## 异或运算（XOR）的重要性质

阐述：若存在一个随机变量$y$，和一个独立**均匀**随机变量$x$，此时$x$和$y$的异或（无论$y$以什么方式分布）$z$*总是*一个**均匀的随机变量**。

换句话说如果我取一个随机的分布，然后用独立**均匀**随机变量和它进行异或运算，最终得到的*一定*是一个**均匀随机变**量，即：

证明现不列出。可能会在作者博客上给出。

# 生日悖论（The birthday paradox）

## 定义

**定义**：假设在全集$U$中选择几个独立、以相同方式分布的随机变量$r_i$ ，则若选择$\sqrt{\|U\|}$次，则基本上有很大的几率（超过$\frac{1}{2}$）会有两个相同的元素。换句话说，若抽样$\sqrt{\|U\|}$次，那么很有可能（超过$\frac{1}{2}$）你的两个抽样会是一样的，即：

$$
r_1,...,r_n\in U\mathtt{\quad are\;\:indep.\;\:and\;\:has\;\:same\;\:way\;\:of\;\:distribution}\\
\mathtt{when}\quad n=1.2 \times \sqrt{|U|} \quad \mathtt{than}\quad Pr[\exists i\neq j: \;r_i=r_j ]\ge \frac{1}{2}\\
\text(\exists \quad \text{means "exist")}
$$

> 该定理被叫做“悖论”是因为经常使用该定理来描述人们的生日。因为根据该定理，最少仅仅需要$1.2\times\sqrt{365}$个人聚集在一起就能够使得有两个人有同样的生日的概率大于$\frac{1}{2}$，即24个人。这就是为什么叫做悖论，因为24应该是一个比大多数人的预计都要小的数。

## 应用

![slide-1](http://ray-eldath.image.alimmdn.com/post/hello-cryptography-3/slide-1.png)

所以只要取样个数超过全集大小的平方根，抽到相同元素的概率将会很快趋近于$1$。

以后我们会再返回并更具体地学习生日悖论，但目前就到此为止。

# 结语

本节和上一节主要介绍了概率分布基础、事件、随机变量基本知识以及一种在密码学中非常常用也非常重要的算法XOR（异或运算，$\oplus$）及其基本性质，最后简要介绍了密码学中一个非常中心的理论：生日悖论。

**本节和上一节的知识相当基础和重要。**

下一节/下一章我们会开始学习我们的加密系统的第一个例子。

## 本节完 本章完
