---
layout: post
title: 你好，密码学(7)：流密码基础：定义及其应用（续）
date: 2018-06-08
categories: 技术
tags: 技术 你好，密码学 系列文章
cover: https://res.cloudinary.com/ray-eldath/image/upload/header/hello-cryptography-7.png
---

流密码基础：定义及其应用（续）

# 现代流密码：Salsa20

在讨论了一个不好的PRG（线性反馈移位寄存器，LFSR）和一个不好的密码（内容扰乱系统，CSS）之后，接下来我们再简要讨论一个设计良好的现代流密码系统：Salsa20。

顺带一提，Salsa20是由[eSTREAM项目](http://www.ecrypt.eu.org/stream/index.html)发布的。

## 构造

Salsa20的PRG并不太符合我们先前介绍的PRG的结构。类似LFSR的PRG要输入一个密钥，随后输出伪随机序列；而类似Salsa20的PRG则要输入一个密钥**和一个新鲜值（Nonce）**，输出伪随机序列。这个新鲜值将参与伪随机序列的生成，用于保证即使密钥相同，输出的伪随机序列依然不同。关于这个新鲜值我们之后还会讲到。

Salsa20输入**128位或256位密钥**和一个**64位的新鲜值**，每轮产生64字节（512位）的输出。

实际上，Salsa20每轮的计算还与一个**轮计数器**有关。该计数器以`0`为初始值，逐轮递增。由于Salsa20每轮使用轮计数器的值不同，因而最终可以输出极长的伪随机序列。

## 操作

Salsa20的运算由若干轮组成，每轮执行一些运算，产生64字节的输出，最后将这若干轮个64字节拼接得到最终的输出。

### 每轮

接下来我们来看看Salsa20**每轮**的基本操作：

1. **初始化。**Salsa20初始状态长64字节，每轮也输出64字节。这64字节如此初始化：

   ![img](https://res.cloudinary.com/ray-eldath/image/upload/post/hello-cryptography-7/Salsa20%20-%201.png)

   > 图中的`B`表示“字节”，以下均同

   其中的$\tau_0$、$\tau_1$、$\tau_2$和$\tau_3$分别表示三个不同的固定常数，它们的值由Salsa20规范给定。

2. **重复施加函数。** 对上述初始状态施加一**可逆的**循环移位函数$h$ ，重复约12轮。

   ![img](https://res.cloudinary.com/ray-eldath/image/upload/post/hello-cryptography-7/Salsa20%20-%202.png)

   其中函数$h$为一个*高效、可逆*的函数以保证Salsa20符合一致性方程。关于函数的详细信息可参见：[Salsa20 - Design, Specification, Security and Speed](http://www.ecrypt.eu.org/stream/p3ciphers/salsa20/salsa20_p3.zip)

3. **逐字加。**最后，将64字节的初始状态（上图橙框）与施加了位移函数的数据（上图红框）进行4字节一次的加法，得到该轮的输出。（这就不用上图了吧Orz）

事实上，如果没有最后一步*逐字加*，Salsa20将没有*任何的*伪随机性；因为：

- 我们已知输出和输入的结构
- 函数$h$完全可逆
- 故而，在没有逐字加这一步的情况下，还原原始输入，进而得到密钥$k$会很容易。

### 总操作

以下给出Salsa20的总计算式：

$$
\mathtt{Salsa20(}k;\;r\mathtt)\::=\:H(\:k,\;(r,\:0)\:)\quad||\quad H(\:k,\;(r,\:0)\:) \quad || \quad ...
$$

其中：

- 每个由拼接符号$\|\|$分隔的即为每轮的输出，里边的数字代表的正是轮计数器。
- 函数$H(...)$表示“轮”，具体操作步骤在上边有给出。

可以发现，每轮的输出依赖于每轮的初始状态，而每轮的初始状态依赖于轮计数器，这意味着只要轮计数器不同，在执行了若干次循环位移函数后，每轮的输出将会相当不同。

而根据定义可知，轮计数器是逐轮递增的，显然每轮都不同，因而每轮的输出也不会相同了。故而将每轮的输出拼接后得到的总输出具有相当的伪随机性。

> “当你的轮计数器递增，0，1，2，3 ...；它会给你一个伪随机序列，要多长有多长。”

## 分析

> 需要指出，Salsa20有很多不同的版本（Salsa20/5、Salsa20/6、Salsa20/12等），它们唯一的不同仅在于每轮重复施加函数$h$的次数不同。如Salsa20/5每轮重复施加函数$h$5次，Salsa20/12每轮重复施加函数$h$12次，以此类推。

最后，我们再来看看Salsa20的安全性、性能及其影响。

### 安全性

作为流密码，Salsa20似乎是无法预测的。根据[cr.yp.to - Stream-cipher attacks](http://cr.yp.to/streamciphers/attacks.html)中的描述，256密钥版本、每轮重复施加12次函数的Salsa20/12目前仍未被攻破且被认为具有256位的安全性（256-bit security）。其它一些重复施加较少次函数的版本（Salsa20/5、Salsa20/6）则已被攻破。

各不同版本的Salsa20的安全情况见下表：

| 版本       | 安全情况                        |
| ---------- | ------------------------------- |
| Salsa20/5  | 已被攻破（broken）              |
| Salsa20/6  | 已被攻破（broken）              |
| Salsa20/7  | 151位安全性（151-bit security） |
| Salsa20/8  | 249位安全性（249-bit security） |
| Salsa20/9  | 256位安全性（256-bit security） |
| Salsa20/10 | 256位安全性（256-bit security） |
| Salsa20/11 | 256位安全性（256-bit security） |
| Salsa20/12 | 256位安全性（256-bit security） |
| Salsa20/20 | 256位安全性（256-bit security） |

关于Salsa安全性的讨论，另见（引自[维基百科 - Salsa20#密码分析](https://zh.wikipedia.org/zh-hans/Salsa20#%E5%AF%86%E7%A0%81%E5%88%86%E6%9E%90)）：

![img](https://res.cloudinary.com/ray-eldath/image/upload/post/hello-cryptography-7/Snipaste_2018-02-21_17-03-06.png)

### 性能

前文我们提到过，CSS在现代计算机系统上运行得甚至要比一些现代的流密码系统慢上不少。我们来看看是不是真的如此（引自[ECRYPT - revision 206](http://www.ecrypt.eu.org/stream/phase3perf/2007a/pentium-4-a/)，其中数字的含义见[ECRYPT - eSTREAM Optimized Code HOWTO](http://www.ecrypt.eu.org/stream/perf/#performance)）：

> 测试使用eSTREAM testing framework，运行于2.80GHz主频Intel Pentium 4处理器（这是2007年出来的报告）。
>
> 作为对照，以下还列出了AES算法（CTR模式）和RC4的性能信息。

| 密码名称   | 密钥长度（位） | 新鲜值长度（位） | 净加密速度（处理器周期/字节） |
| ---------- | -------------- | ---------------- | ----------------------------- |
| Salsa20/8  | 128            | 64               | 7.60                          |
| Salsa20/12 | 128            | 64               | 10.62                         |
| RC4        | 128            | 0                | 13.07                         |
| AES-CTR    | 128            | 128              | 17.81                         |

> 净加密速度单位为`处理器周期 / 字节`，越小越好。
> 见：[什么是“加密率”？ - 玛尼的回答 - 知乎](https://www.zhihu.com/question/267463624/answer/372855121)
>
> maya这文档看得我超费劲...

还有一份感觉*不那么严谨*的文档列出了有关算法的更易理解的性能参数（引自[Comparison of Encryption Methods' Speed - libQtShadowsocks#Wiki](https://github.com/shadowsocks/libQtShadowsocks/wiki/Comparison-of-Encryption-Methods%27-Speed)）：

> 测试运行于Intel Core i7-6500U，Lenovo YOGA700-14上。
>
> 以下“耗时”项为加密*100MB数据报*消耗的时间，越低越好。

| 密码名称 | 密钥长度（位） | 耗时（ms） |
| -------- | -------------- | ---------- |
| Salsa20  | *未给出        | 263ms      |
| RC4-MD5  | *未给出        | 2640ms     |
| AES-CFB  | 128            | 1860ms     |

可以看出，Salsa20无论在安全性还是在性能方面都是不错的。

# 结语

本文我们主要讨论了一个安全的RPG Salsa20，它的总体架构，它的运行过程，以及对它的一些性能和安全分析。下一节我们将讨论流密码的安全性。

## <完>

> 为什么这篇文章这么短？因为我实在写不下去了...๑乛◡乛๑