---
layout: post
title: 你好，密码学(6)：流密码基础 - 定义及其应用
date: 2018-02-22
categories: 技术
tags: 技术 你好，密码学 系列文章
cover: http://ray-eldath.image.alimmdn.com/header/hello-cryptography-6.png
---

# 回顾 & 导言

## 上节回顾

在上一节中，我们花很长篇幅讨论了PRG——伪随机数生成器，本节我们将看到作为流密码核心的PRG如何发挥效用，以及几个PRG的实现和由此构建的流密码系统。

## 快速回顾

我们先来快速回顾一下前边几篇文章中讲到的相关要点：

### OTP

首先，我们定义了**一次性密码本（OTP）**密码：

$$
E(k,\:m)=m\oplus k\qquad D(k,\:c)=c\oplus k\quad(\;\|k\|=\|m\|\;)
$$

OTP密码的加密和解密都仅仅是做了一个简单的异或运算，但它却具有**完美安全性**：密文不透露有关明文的*任何*信息。但是，纵使这是一种完美安全的密码，它却无法实现，因为香~~浓~~农曾证明过以下定理：

**若一个密码完美安全，则其全体密钥的数量的必须不少于全体明文的数量**

因此，OTP密码完美安全，因为其$\|k\|=\|m\|$；但是它又不可实现，因为若存在一种能安全运输*和密文一样长的密钥*的方法，那我为何不直接用这种方法安全运输明文呢？

因此，像OTP密码一样要求密钥和明文一样长的密码，是无法实现的。

### PRG

伪随机数生成器（PRG）是一个**高效**的、**确定**的、**不可预测**的函数$G$，能将一个**长度为$s$的二进制数字符串**，扩充为一个**长得多的、长度为$n$的二进制数字符串**。

这个函数是否不可预测将直接决定它是否安全，也将决定以它作为核心的流密码是否安全。

### 流密码

为解决OTP不可实现的问题，流密码引入了一个*不可预测*的PRG$G$，将输入的密钥$k$（又称为“种子”，我们将在下文中交替使用这两种称法）送入$G$，从而得到一个**和明文一样长的**伪随机密钥$G(k)$，再使用这个伪随机密钥完成加密过程。即：

$$
E(k,\:m)=m\oplus G(k)\qquad D(k,\:c)=c\oplus G(k)\quad(\;\|\:G(k)\:\|=\|m\|\;)
$$

这就解决了OTP不可实现的问题。因为输入的密钥$k$的长度显然是远远小于明文的长度的，只是使用PRG$G$将其进行扩充而已。

由此可见，正如我们反复强调的那样，一个流密码的安全性将完全依赖于它使用的PRG的安全性，也即这个PRG的不可预测性。

> 关于上文我们反复指出流密码的安全性等同于其不可预测性，这里作如下解释：
>
> Andrew C. Yao在其于1982年发表的论文《Theory and application of trapdoor functions》中证明了：
>
> **一个不可预测的PRG是安全的。**
>
> 事实上，上述定理的逆定理（一个安全的PRG必然不可预测）也很容易证明。在此就不给出证明过程。
>
> 关于以上两个定理的更多讨论，请见：
>
> - [StackExchange - An unpredictable PRG is secure (Theorem Yao'82)](https://crypto.stackexchange.com/questions/18043/an-unpredictable-prg-is-secure-theorem-yao82)
> - [ResearchGate - Theory and application of trapdoor functions](https://www.researchgate.net/publication/4355261_Theory_and_application_of_trapdoor_functions)

# 线性反馈移位寄存器（LFSR）

在开始介绍流密码之前，我们先介绍一种简单的PRG：线性反馈移位寄存器（LFSR）。我们下面将要介绍的一些流密码都使用这种PRG作为其算法的核心。这种PRG理解、实现起来都很简单，因而攻破起来也很简单（嗯？）

## 操作

以下简要介绍LFSR运行的基本操作：

1. **初始化。**首先，LFSR会以给定的种子（即密钥）初始化寄存器，以位（比特）为单位：
  ![LFSR - 1](http://ray-eldath.image.alimmdn.com/post/hello-cryptography-6/LFSR+-+1.png)

2. **抽头。**随后，LFSR从这些比特位中随机选取一些（称作“抽头”），并计算这些抽头的异或：

  ![LFSR - 2](http://ray-eldath.image.alimmdn.com/post/hello-cryptography-6/LFSR+-+2.png)

  图中绿色的即为抽头。橙色的为将所有抽头异或的结果。

3. **位移并附加。**最后，LFSR将这些抽头异或的结果附加到寄存器开头，并使寄存器向右位移，最后一个比特位作为结果输出：

   ![LFSR - 3](http://ray-eldath.image.alimmdn.com/post/hello-cryptography-6/LFSR+-+3.png)

> 用ProcessOn做这些插图累死了... ...（´□｀川）
>
> 顺便说下：上边图中看上去非常丑的“⊕ XOR”感觉是ProcessOn的bug...

很简单吧？初始化寄存器后，每个时钟周期LFSR将执行重复上边这些步骤，选取一些抽头，将它们异或，随后寄存器右移输出最右一位并将异或结果附加到寄存器开头。

## 好处 坏处

使用LFSR的好处是显然的：**简单还快！**计算设备能以非常快的速度运算异或，再加上很短的运算步骤，LFSR生成伪随机数序列的速度将会很快。同时我们可以看出，LFSR显然是面向硬件设计的，这使得使用硬件实现高速运行的LFSR将会相当方便和简单（见[NEW WAVE Instruments - LFSR Reference#LFSR Generator Implementations](http://www.newwaveinstruments.com/resources/articles/m_sequence_linear_feedback_shift_register_lfsr.htm#LFSR%20Implementations)）。

**但是**，“由于寄存器的状态是有限的，它最终肯定会是一个重复的循环。”（摘自[维基百科 - 线性反馈移位寄存器](https://zh.wikipedia.org/wiki/线性反馈移位寄存器#firstHeading)）因而，LFSR并不安全。使用LFSR构建的流密码系统也不会安全。事实上，几乎所有使用LFSR构建的流密码系统（即使它使用了四个LFSR，见[Wikipedia - E0 (cipher)#Cryptanalysis](https://en.wikipedia.org/wiki/E0_(cipher)#Cryptanalysis)）都已被攻破。

关于LFSR安全性的讨论我们暂时就到这里。下边我们还会在对一个使用了LFSR的流密码进行分析的过程中再次探讨LFSR的安全性。

# 内容扰乱系统（CSS）

接下来，我们将讨论一个简单的流密码——用于加密DVD的内容扰乱系统（Content Scramble System，CSS）。这个系统使用了两个LFSR，但即便如此，它也已被攻破。事实上，这个流密码不仅不安全，在现代计算机上运行起来也比那些安全的流密码慢上不少。这点我们之后还会讲到。

## 构造

CSS使用5字节（也就是40比特）长度的密钥和两个LFSR：一个长17比特的LFSR（即其寄存器长17比特）和一个长25比特的LFSR。每轮两个LFSR运行几轮，最后输出一个字节。

## 操作

我们先来看看CSS的基本操作：

1. **初始化。**CSS使用$1$接上5字节种子的前两个字节初始化17比特的LFSR，再用$1$接上5字节种子的后三个字节初始化25比特的LFSR：

   ![CSS - 1](http://ray-eldath.image.alimmdn.com/post/hello-cryptography-6/CSS+-+1.png)

   图中$\|\|$表示“拼接”。这个符号在之前的一些文章中也有出现。

2. **运行。**CSS将每个LFSR都运行八轮。

   ![CSS - 2](http://ray-eldath.image.alimmdn.com/post/hello-cryptography-6/CSS+-+2.png)

3. **求和后求模，得到一个字节。**最后，CSS将两个LFSR的输出求和并对256求模，得到一个字节。本轮结束。

   ![CSS - 3](http://ray-eldath.image.alimmdn.com/post/hello-cryptography-6/CSS+-+3.png)

总的来说，CSS初始化后每轮运行每个LFSR各8轮（产生八个输出），随后将这些输出求和并模256，得到一个字节。

CSS是一个相当简单的密码。易于硬件实现、速度快，在廉价设备上依能很好地工作。故在廉价设备上也能很好的破解。

## 攻破

下面我们将简要讨论对CSS的攻破。

以下讨论基于一段密文、一段很短的已知的明文头部（如MPEG-4要求以特定的“box”数据开头，见[MP4文件格式的解析，以及MP4文件的分割算法](http://www.cnblogs.com/haibindev/archive/2011/10/17/2214518.html)），我们能通过最多$2^{17}$次尝试还原出完整的明文。

### 操作

1. 首先，我们将密文与已知的明文头部异或，可以得到一段很短的CSS输出头部（图中青色部分）：

   ![CSS Crack - 1](http://ray-eldath.image.alimmdn.com/post/hello-cryptography-6/CSS+Crack+-+1.png)

   为方便说明，下边我们都假设已知的明文头部为20字节，得到的CSS输出也为20字节。

2. 随后，我们**猜测**第一个17比特长度LFSR的初始状态，运行它以输出20字节，再用先前得到的CSS输出头部减去这个LFSR的输出，根据CSS的定义，这时我们得到的这个字串（可能）是25比特长度LFSR的一个可能输出。

   ![CSS Crack - 2](http://ray-eldath.image.alimmdn.com/post/hello-cryptography-6/CSS+Crack+-+2.png)

3. 事实上，判断一个20字节的字串是否来自一个25比特长度的LFSR是简单的。这里将包含一个循环判断逻辑：

   - 如果我们判断上一步得到的字串确实是25比特长度LFSR的一个可能输出，则说明我们对17比特长度LFSR的初始状态猜测正确。
   - 如果不是，则说明对17比特长度LFSR的初始状态猜测错误，那我们就再猜一次。

4. 随后我们再用上一步得到的25比特长度LFSR的一个可能输出逆推出25比特长度LFSR的初始状态。


这样，通过最多$2^{17}$次尝试，我们就能还原出所有LFSR的初始状态，并运算出余下的CSS输出，最终完成解密。

在家用计算机上，使用类似上述操作的CSS破译器（如DeCSS，见[DeCSS Central - CSS and DeCSS](http://www.lemuria.org/DeCSS/decss.html)）通常能在一天之内破译任意一部使用CSS加密的影片。因而现在使用CSS加密DVD已经不流行了。

## 结语

先前我们已经提到，LFSR并不安全，从上边“判断一个20字节的字串是否来自一个25比特长度的LFSR是简单的”这句话也可看出。使用LFSR的CSS因LFSR的不安全而被攻破，其它使用LFSR的流密码系统也是如此。

# 结语

本节中，我们讨论了一个并不强健的PRG（LFSR）和基于这个PRG的同样并不强健的流密码系统（CSS），这些密码系统均已被攻破。在下一节，《流密码基础：定义及其应用（续）》中，我们将讨论一个现代的流密码系统：Salsa20。

> 其实关于Salsa20的玩意已经差不多了的...这里不一起发出来主要是因为现在的内容已经很多了...而且...[我有一点疑问](https://www.zhihu.com/question/267463624)...

## <完>