---
title: "Self：编程语言的变革仍未到来"
date: 2021-03-15 15:09:20
cover: /programming/self-language/cover.jpg
thumbnail: /programming/self-language/cover.jpg
toc: false
tags:
 - 程序设计范式（paradigm）
 - 程序设计语言
categories: programming
---

> 本文是一篇*小作品*。

本文引用并翻译了在 HOPL III 上发表的论文 *Self*（DOI: [10.1145/1238844.1238853](https://doi.org/10.1145/1238844.1238853)）中的倒数第二节。在 ACM Digital Library 收录的所有 [HOPL（*History of Programming Languages*）](https://dl.acm.org/conference/hopl) 论文中，这篇论文的下载量排到了第八位。

我将要引用并翻译的段落出自论文的倒数第二节 *“8. Conclusion”（8. 结论）* 。在读完全文后，我在我的私人群里如此表达我的感受：

<!-- more -->

![](image-20210315133343698.png)

![](image-20210315133410399.png)

首先这里简单用几句话概括一下 *Self*。*Self 语言* 是一门*基于原型的（prototype-based）*、*纯消息传递（pure message-passing）*的*无类式（classless）*面向对象语言。这门语言去除了*类*和*对象*之间的分立，通过灵活的原型链机制实现了*实现复用*（reuse of implementation，即*继承*），并且*彻底贯彻*了*消息传递*，抹除了*赋值（assignment）*和*计算（computation）*之间的界限——简单性（simplicity）、可理解性（comprehensibility，尽可能贴近现实世界）和统一性（uniformity）是该语言一贯坚持的原则。尽管资助该语言的计划早在 1995 年就被迫中止，Self 依然十分深刻地影响了当今世界上最活跃、最重要的几大编程语言：JavaScript 几乎和 Self 如出一辙，同样抹去了类（class）的存在，同样基于原型链达成了OOP（然而之所以促使 Self 作出这些设计*结果*的*重要原则和思想*在如今的 JavaScript 中已经完全得不到体现——这无疑十分可惜和遗憾）；而 Self VM 的实现技术非常直接地影响了 JVM 的实现——诸如 polymorphic inline cache, dynamic optimization & deoptimize, delay compilation of uncommon cases, tiered compilation & on-stack replacement 等技术，在现代 JVM （甚至是所有的现代*高级语言虚拟机*）中都起着核心作用。

我希望各位在读完本文的节选和翻译后也能有相同的体会，那就是尽管这门语言*早已死亡*，但这却丝毫不影响语言本身和语言的设计者们的伟大——他们是真正对程序、对程序的*可理解性*和*与现实世界的一致性*保持高度关注和热诚的人们。如今，正如我在前一篇博文中提到的，计算机领域的抽象越来越高级，实现越来越复杂；编程语言本身包含越来越多的特性和抽象——这些抽象距离我们真实可感的现实世界越来越远、越来越远，我们要问，如今的编程语言是否早已偏离了它本应秉持的方向和价值？

本文的标题一方面是引用自论文作者论文摘要（Abstract）的最后一句 *Nevertheless, the vision we tried to capture in the unified whole has yet to be achieved.* ，另一方面（更主要的）是在向 Alan Kay 著名的演讲 **The computer revolution has NOT happened yet** 致敬——Alan Kay 的这个演讲是编程语言顶会 OOPSLA 1997 年的keynote，被 Joe Armstrong 认为是 “80个计算机领域必学思想” 中的一个。

本文中提及的各种材料的出处和获取方式详见文末。译文附在原文之后。

## 正文

> **8. Conclusion**
>
> Shall machines serve humanity, or shall humanity serve machines? People create artifacts that then turn around and reshape their creators. These artifacts include thought systems that can have profound effects, such as quantum mechanics, calculus, and the scientific method. In our own field thought systems with somewhat less profound effects might include FORTRAN and Excel. Some thought systems are themselves meta-thought systems; that is, they are ways of thinking followed when building other thought systems. Since they guide the construction of other thought systems, their impact can be especially great, and one must be especially careful when designing such meta-thought systems. 
>
> We viewed Self as a meta-thought system that represented our best effort to create a system for computer programming. The story of its creation reveals our own ways of thinking and how other meta-thought systems shaped us [US87, SU95]. We kept the language simple, built a complicated virtual machine that would run programs efficiently even if they were well-factored, and built a user interface that harnessed people’s experience in dealing with the real word to off load conscious tasks to precognitive mental facilities. We did all of this in the hope that the experience of building software with the Self system would help people to unleash their own creative powers. 
>
> However, we found ourselves trying to do this in a commercial environment. Free markets tend to favor giving customers what they want, and few customers could then (or even now) understand that they might want the sort of experience we were creating. 
>
> Years later, the Self project remains the authors’ proudest professional accomplishment. We feel that Self brought new ideas to language, implementation, programming environment, and graphical interface design. The original paper [US87] has been cited over 500 times, and (as previously mentioned) received an award at the OOSPLA 2006 conference for among the three most influential conference papers published during OOPSLA’s first 11 years (1985 through 1996). Self shows how related principles can be combined to create a pure, productive, and fun experience for programmers and users.
>
> **So, what happened? Why isn’t your word processor written in Self? While we have discussed the struggle of ideas that gave birth to Self, we have not addressed the complex of forces that lead to adoption (or not) of new technology. The implementation techniques were readily adopted, whereas semantic notions such as using prototypes, and many of the user interface ideas behind Morphic, were not so widely adopted. We believe that, despite the pragmatic reasons mentioned in section 6.1.5, this discrepancy can better be explained by the relative invisibility of the virtual machine. If there are dynamic compilation techniques going on underneath their program, most users are unlikely to know or care. But the user interface framework and the language semantics demand that our users think a certain way, and we failed to convince the world that our way to think might be better. Did our fault lie in trying to enable a creative spirit that we mistakenly thought lay nascent within everybody? Or are there economic implications to the use of dynamic languages that make them unrealistic? Many of us in the programming language research community secretly wonder if language research has become irrelevant to most of the world’s programmers, despite the obvious truth that in many ways, computers remain painful, opaque black boxes that at times seem intent on spreading new kinds of digital pestilence.**
>
> **Almost two decades after the conception of Self, the imbalance of power between man and machine seems little better. We are still waiting for computers to begin to live up to their full promise of being a truly malleable and creative medium. We earnestly hope that Self may inspire those who still seek to simplify programming and to bring it into coherence with the way most people think about the real world.**

译文如下：

> **8. 结论**
>
> 是机器服务于人类，还是人类服务于机器？人们创造了人造物，而它们又反过来重塑它们的创造者。这些人造物包括能产生深远影响的思想体系，如量子力学、微积分和科学方法。在计算机领域中，影响不那么深远的思想体系包括 FORTRAN 和 Excel；有些思想体系其实是元思想体系——也就是说，它们是构建其他思想体系时应该遵循的思维方式。由于它们指导着其他思想体系的构建，所以它们的影响尤其大，人们在设计这样的元思想体系时必须特别小心。
>
> 我们把 Self 看作一个元思想体系，它代表着我们为创造一个计算机编程系统所做的最大努力。创造 Self 的故事揭示了我们自己的思维方式，以及其他元思想体系是如何塑造我们的 [US87, SU95]。我们始终保持语言本身的简单，并实现了一个十分复杂的虚拟机，它能够十分高效地执行良好构造的程序；我们还建立了一个用户界面，利用人们处理真实世界的经验，将心智负担转化为对潜意识的利用（从而消除了心智负担）。我们所做这一切努力，是因为我们希望 Self 系统能够帮助人们释放自己的创造力。
>
> 然而，我们发现我们是试图在商业环境中做这件事情。自由市场的规则要求我们向客户提供他们想要的东西——然而在当时（甚至是现在），只有很少的客户才能认识到他们或许需要我们创造的那种体验。
>
> 近半个世纪过去，Self 项目仍然是作者们最为自豪的专业成就。我们坚信 Self 为语言设计和实现、编程环境和图形界面设计带来了启发。我们最初提出 Self 的那篇论文 [US87] 被引用了超过 500 次，在 OOSPLA 2006 会议上获得了*历史最具影响力论文奖*——这一奖项授予 OOPSLA 会议的头11年（1985年到1996年）间发表的前三篇最具影响力的论文。Self 展示了如何将相关原则结合起来，从而为程序员和用户创造一个纯粹的、高效的，以及有趣的体验。
>
> **那么，到底发生了什么？为什么你的文字处理器不是用 Self 写的？尽管我们已经讨论了催生出 Self 语言的种种思想斗争，但我们还没有讨论导致新技术被采纳（或不被采纳）的复杂力量。（正如 Self VM 的实现技术很快就广泛渗透到了当今世界上几乎所有的高级语言虚拟机中）实现技术很容易被采纳，而语义概念——如使用原型，以及 Morphic 背后的许多用户界面理念——却没有被广泛采纳。我们认为，尽管我们在 6.1.5 节中已经列举了种种实用性原因（如内存要求过高），这种差异或许可以用虚拟机的相对不可见性来更好地解释：我们编写的程序之下正在不可见地执行着动态编译，而大多数用户其实并不怎么了解或关心它们。但是，用户界面框架和编程语言语义却要求用户以某种特定的方式思考，而我们却未能说服世界，以我们提出的方式思考要更好些。我们是错在试图唤醒事实上并不存在于每个人心中的创新性精神吗？还是说使用动态语言有一些经济学意义上的影响，使之不切实际？我们，编程语言研究界的许多人们都在暗自怀疑，编程语言研究是否已经与世界上大多数的程序员无关——尽管显而易见的事实是，在许多方面，计算机仍然是痛苦的、不透明的黑盒子，它仍在传播新的数字瘟疫。**
>
> **在 Self 的种种理念提出近20年后，人与机器之间的力量不平衡似乎并没有什么好转。我们仍在期待计算机开始实现使其自身成为真正可塑的创造性媒介的全部承诺。我们真诚地希望，Self 能够激励那些仍在努力寻求简化编程的方式，使之与大多数人思考现实世界的方式相一致的人们。**
>
> **（编程语言的变革仍未到来。）**

## 主要引述来源

- David Ungar and Randall B. Smith. 2007. Self. In <i>Proceedings of the third ACM SIGPLAN conference on History of programming languages</i> (<i>HOPL III</i>). Association for Computing Machinery, New York, NY, USA, 9–1–9–50. DOI:https://doi.org/10.1145/1238844.1238853
- Ray Eldath. 计算机领域的三个重要思想：抽象，分层和高阶. this blog.
- Alan Kay at OOPSLA 1997 - The computer revolution hasn't happened yet. https://youtu.be/oKg1hTOQXoY
- GOTO 2018 • Computer Science - A Guide for the Perplexed • Joe Armstrong. https://youtu.be/rmueBVrLKcY