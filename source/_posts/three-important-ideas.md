---
title: "计算机领域的三个重要思想：抽象，分层和高阶"
date: 2021-03-06 02:53:11
cover: /img/three-important-ideas.jpg
thumbnail: /img/three-important-ideas.jpg
toc: true
tags:
 - 随笔
 - 程序设计范式（paradigm）
categories: programming
---

昨晚看了点比较有意思的东西，于是决定写一篇文章简单讲一下。

本文致力于概括本寒对计算机界三个重要思想的体会和认识。我希望做的并不是简单的百科全书式的列举（“`A` 体现了抽象思想；`B` 体现了分层思想…”），而是从这些思想中选取几个我个人**较有体会**（或者是我单纯觉得**十分有趣**）的侧面拿来细讲。这些侧面仅仅能覆盖这些思想应用范围中十分微小的一部分，它们**并不是**最有代表性的、**亦非**最为重要的——仅仅因为，我个人对这点侧面有些体会，或者我个人认为它比较有趣而已。

同样需要强调，和本博客大多数文章一样，本文**个人意见色彩浓厚——本文并不客观，更绝不权威。**

为方便行文，文中提到的大量引述来源不会标号、也不会在文中注明。关于攥写时参考的资料，请参见文末 “参考文献”。

<!-- more -->

## 有关抽象

要从软件开发的所有指导性思想（或者说是哲学）中找到最容易想到、最广为人知的那一个，那必定是本节的主题：**抽象**。

这是一种重要的心智活动，它从具体的、实际的概念和经验中提炼出较不具体的、更加泛化的部分。在软件生产实践中我们无时无刻不在使用这一工具：我们从具体的*类*中提炼出*接口*；从实在的*数据*中提炼出*数据结构、泛型和多态*；从生动的*现实场景*中提炼出*实体*并为之命名（有时候相比*提炼出抽象*而言命名还要更加困难）。某种程度上，本文中提到的其它两点思想*分层*和*高阶*同样是抽象的一种形式，但是我个人认为将它们单独拎出来会有一些特别的指导性意义和方法论层面上的价值——关于这点在本文的其它部分中会再作说明。

关于*纯粹的抽象*这一点，我主要想特别谈谈*数学抽象*。众所周知，这一类概念在计算机界中有重要应用。

### “数学抽象”

长久以来（我不知道这是否是我个人的经验，我希望有和我一样有相同看法的朋友），我对*编程语言理论 PLT*总有一种现在看来是错误的认识，那就是以为*抽象代数是 PLT 的先修课*。一到两年前我（极其痛苦，现在看来却同时是**相当无用**地）啃过丘维声的《抽象代数讲义》；粗略且不完整地看过油管上 Bartosz Milewski 的猫论视频；曾认真阅读过有关 Scala 中 Cats 库的拆解，并自己实现了其中的很多抽象。

然而，现在看来，这些经历并未帮助我理解很多东西。知道了 Monad 的形式定义（`pure` 和 `flatMap`，同时 Monad 还是一个 Functor 因而具有 `map`）和 Moniod 的抽象定义（幺半群，指一个定义有二元运算的集合，它满足结合律，并且有单位元）并没有帮助我更好地理解程序的意义和结构。现在看来这些知识几乎是独立的小块——它们与我绝大部分的编程经验并没有联系起来，只有极少数的时候（比如现在要写博文了= =）才会想起来这点东西。

于是我开始思考，**偏向工业的程序员们应该如何对待抽象概念（尤其是从数学界借用的）？**学习这些抽象概念到底能不能（以及如何能）提升一般程序员、或者是计算机产业界对代码的理解和认识？学习这些抽象概念，我们是否能籍以此成为更好的程序员呢？

我想我对于这个问题不是太有发言权，于是我给 Dan Grossman 写了一封邮件——这位 Dan Grossman 大概是 PLT 学界（和教学界）的重要人物（某种程度上，因为听过他在 Coursera 上的系列课程 Programming Languages，也是我的 *“技术偶像”*（我一直在找一个不那么中二不那么尴尬的词，但是一直没有找到，如果你有什么比较好的建议非常欢迎在评论区告诉我… :-|），我简单翻译了下这封邮件，其中从 Bjarne 的一篇论文（见文末）中他和 Alexander Stepanov 之间的 “私人通讯” 里引用了一段话 ：

> Dr. Dan:
>
> （客套话略）
>
> I’m here to ask you a question that has confused me for a while. The story may be a bit long, and I’m genuinely sorry for that. Many of my friends have been devoted themselves to PLT for many years. And, as their friend, I noticed there is a deep connection between PLT and mathematics — especially those overwhelmingly perplexing and abstract subdisciplines, e.g. Abstract Algebra. As a programmer, I’ve also noticed that there’re many (and even more and more) articles are intended to introduce some algebraic concepts to programming, e.g. Monad, Functor. Most of these articles use some analogy like a box, funnel, to make them more understandable. 
>
> One of my friends has the following comment on these articles: he said that the only way to truly understand abstraction, is through abstraction, and inaccurate analogy cannot help. He said that to effectively (or even usefully) learn these abstractions, you have to learn mathematics, not some rough material, all it has is analogies.
>
> This trend — perplexing words used to boast, perplexing explaining articles read to have nothing — is not good. Personally speaking, I think these abstract concepts are just not useful. **Just** knowing a Promise in JavaScript is a Monad per se, and Functor “is a box” could NOT help you being a better programmer. Using these words in your own library makes the newcomer feel uncomfortable.
>
> **So, here’s my question:** What’s your opinion towards these abstractions, many of them are quoted directly from abstract mathematics? Can learn these abstractions helping me be a better programmer? And if I just “know” them but not “understand” them in a more abstract manner (which is the Abstract Algebraic), can I still be a better programmer? Would you recommend a “Joe Coder” to learn or use these words during his career? Will you write some easy-to-read articles to make “Joe Coder” learn them by showing them bunches of analogies?
>
> My personal answer is No. And here I quoted a paragraph from Bjarne’s paper and is said by Alexander Stepanov:
>
> > At that time I discovered the works of Euler and my perception of the nature of mathematics underwent a dramatic transformation. I was de-Bourbakized, stopped believing in sets, and was expelled from the Cantorian paradise. I still believe in abstraction, but now I know that one ends with abstraction, not starts with it. I learned that one has to adapt abstractions to reality and not the other way around. Mathematics stopped being a science of theories but reappeared to me as a science of numbers and shapes.
>
> As a personal answer I’d like to say that reading those articles, just “knowing” them won’t help you become a better programmer. And using those words in libraries makes things even **worse**. A programmer should be honored if and only if they use and write things technical superior, not something mathematical superior. 
>
> And this is my question. I’m eagerly waiting for your reply, as I think your opinion could really help me being a better programmer, and should I learn these concepts, how can learning them help me (if it could).
>
> ---
>
> 我谨在此向您提出一个疑问，这个疑问困扰我已经有一阵了。故事大概会有些长，而我对此十分抱歉。我的很多朋友已经在 PLT 界摸爬滚打了多年，作为他们的朋友，我注意到 PLT 和数学——特别是那些（于我而言）尤其难搞和抽象的学科，比如抽象代数——有着深刻的联系；而作为一位程序员，我又注意到如今有很多（以及越来越多的）文章致力于将一些代数的概念引介到编程中，比如 Monad，比如 Functor；这些文章大多会使用一些类比，比如盒子 box，比如漏斗 funnel，来使这些概念更易理解。
>
> 我的一位朋友如此评论这些文章（如果我作了不确切的引用，我在此对他真诚地道歉）：他说真正理解抽象的唯一方式是通过抽象*本身*，而不是通过并不准确的类比。他说为了充分地（甚至是有用处地）学习这些抽象，你必须去学数学，而不是通过一些糊里糊涂的文章，它们除了类比还是类比。
>
> 这样的一种趋势——令人困惑的词语用来故弄玄虚，令人困惑的文章读来一无所有——并不好。就我而言，我认为这些抽象概念并不大有用。**仅仅**知道 JavaScript 里的 Promise 本质上是 Monad，而 Functor “是一个盒子 box” 并*不能*帮助你成为更好地程序员，而在你自己的库中使用这些词语只会让后来者觉得不舒服。
>
> **所以，这就是我的疑问：**您对于这些抽象——它们中的很多直接引用自抽象的数学分支，有什么看法？学习这些抽象能否帮助我们成为更好的程序员？如果仅仅只是 “知道” 它们但却未以一种更加抽象的视野（即抽象代数）“理解” 它们，我们是否仍然能成为更好的程序员呢？您是否会推荐一位 “大众程序员” 去学习或使用这些东西？
>
> 我个人的回答是，它们不能。在这儿我引用 Bjarne 的论文中的一段话，它们来自 Alexander Stepanov：
>
> > **当时我发现了欧拉的作品，我对数学本质的认识发生了巨大的转变。我被去布尔巴克化了，不再相信集合，被逐出了康托尔的天堂。我仍然相信抽象，但现在我知道，一个人经抽象走向末路，而不是自抽象开启旅途。我知道了，人必须使抽象适应现实，而不是反其道而行之。数学不再是一门理论的科学，而是作为一门数字和形状的科学重新出现在我的面前。**
>
> 我个人的回答是，阅读这些文章，仅仅 “知道” 它们并不能帮助你成为更好的程序员，而在库中使用这些词只会让现状恶化。一个程序员应该以（且只以）使用或编写了技术上更佳的东西为荣，而不是数学上更佳的东西。
>
> 所以这就是我的疑问，而我急切地期待着您的回复。我觉得您的意见可以帮助我成为更好的程序员，以及我是否应该学习这些概念，它们对我能有什么帮助（如果有的话）。
>
> ---
>
> **Have a nice day!**
> Ray Eldath
> GitHub: github.com/Ray-Eldath MyBlog: han.ninja

数日之后（非常惊喜又不大意外地）收到了 Dan 的回邮。他在回邮中说：

> Ray,
>
> These are good but unanswerable questions as it is easy to end up down the path of "what does it mean to learn or understand something" which is a question of consciousness well beyond my expertise. :) For my own worldview, I am a fairly die-hard "operational semantics" person which comes through in my teaching. I believe you can understand monads and functors for programming purposes in terms of how they compute -- what the rules are for operation's like a monad's bind operator. Yes, there are algebraic laws that should be followed, but to understand "what code does" these are not necessary. Entities like monads are very abstract and general -- it's a concept that describes a broad set of programming patterns / data structures / libraries / APIs and the power is that this is a guiding principle for how to both design and use such abstractions. The connection to advanced mathematics has value and helped develop these powerful abstractions, but it is not my view that effective programmers need to understand the full connection. Others will, naturally, vehemently disagree. :-)
>
> ---
>
> 这些都是很好的问题，但我无法回答，因为很容易就会走上 “学习或理解某件事情意味着什么” 的道路，而这是一个意识问题，远远超出了我的专业知识 :)。对于我自己而言，我相当顽固地秉持着 “操作语义”，这一点在我的教学中得到了体现。我相信你可以通过 Monad 和 Functor 在编程目的下的计算方式来理解它们——像是 Monad 的 `bind` 操作符受制于什么规则。是的，有一些代数法则是应被遵循的，但要理解 “哪些代码遵循它们” 并不是必须的。像 Monad 这样的概念是非常抽象和通用的：它是一个描述了一套广泛的编程模式、数据结构、库和 API 的概念，其强大之处在于它们是对如何设计和使用这种抽象的指导原则。与高级数学的联系是有价值的，并且有助于发展这些强大的抽象，但我认为成为高效的程序员并不需要理解全部的联系。其他人自然会强烈反对 :-)。
>
> Best,
> Dan

我想我对这一特别抽象（即数学抽象）的看法，以及一位代表性学者（实话说，我觉得 Dan 并没有确切地回答很多我觉得应该得到回答的疑问）的看法在这一个邮件的来回中已经充分表达。我想你大概已经看出来了，我并不擅长理解抽象——或许这就是我 “动机不纯” 的把柄，所以我要坦诚地承认这一点。我知道有太多太多的人远比我擅长理解抽象，并确实深刻地理解它们的意义和价值。或许他们会走向和 Alexander Stepanov 相反的道路，或许他们也会是 Dan 提到的 “will vehemently disagree” 的 “others”。

## 有关分层

相比*抽象*侧重于一种**心智活动**（从特化的到泛化的，从具体的到笼统的）、强调一种 **“抽取”** 的动作，分层要更加客观，它强调概念之间存在明显的、更加客观的断层，这些断层，或者说是**界面**，或者说是**接口**，区隔了层与层，提出了表面的*接口 / 契约*和深入的*实现细节*间的差别：上一层只需要依赖下一层的*接口*而非更加细致的*实现细节*，这意味着**在良好实现的分层机制中，对于某一层的实现者而言，下一层的绝大部分是不需关心的。**

---

**但是，真的如此吗？**

*抽象*和*分层*向我们一再承诺的 “接口远大于实现细节”、“只需要关心很小的一部分”，在多大程度上是成立的？这样的*封装*有没有被打破的时候？

### “Hyrum's Law”

最早听闻 Hyrum's Law 是在 YouTube 的一个非常著名（评价也非常好）的talk *A Philosophy of Software Design* 里。主讲 John Ousterhout 讲到一半时主持从在线论坛里边抽了个问题问他，其中就提到了这个 Hyrum's Law。

> John Ousterhout 当时还愣了一下问啥事 Hyrum's Law，主持把这个东西的简明定义说了一下，然后附了一句 “actually I Googled this” wwww  看来大家都不是很知道这个东西 😉

这个 Hyrum's Law 的简明定义其实很好理解：

> Hyrum's Law: **With a sufficient number of users of an API, it does not matter what you promise in the contract: all observable behaviors of your system will be depended on by somebody.**
>
> Hyrum's Law: **只要一个 API 有足够的用户，那么无论你在接口契约中承诺什么，你的系统的所有可观测行为都会被某个用户所依赖。**

更容易理解的或许是下边这张 xkcd 的漫画：

[![xkcd#1172](/img/xkcd_workflow.png)](https://xkcd.com/1172/)

*Hyrum's Law*（又作 *The Law of Leaky Abstractions*）描述的是这样一种现象： ***层*被创造出来的本意是通过*分层*这一过程，从而界定一个清晰、明确的边界。这一边界划定了*接口 / 契约*和*实现细节*之间的界限，从而一方面提高了程序员的工作效率（“我只要用它就好啦”），另一方面——更为重要的是——*降低了程序员的门槛，并减轻学习负担*（“既然只要用就好了，那我应该就不用关心底层的实现细节了吧？”）**

然而，**随着一个层的用户逐渐增多，层逐渐失去了*区隔契约和实现细节*的作用：层的每一个可观测行为（不管是不是在契约框定的范围之内）都会被某些用户所依赖，这意味着分层的初衷事实上*已不再有效*：总有一个时候，我们的输入会足够特殊，我们的场景会足够复杂，以致于我们会依赖层的某一个*契约之外*的可观测行为。这个时候，层对我们而言已经*毫无意义*：我们依赖的*早已不是*层创设出的接口 / 契约，而是层的*实现细节本身*。**

这样的例子数不胜数。比如，说到分层，大家脑海中的第一个想法，很可能会是计算机网络以及深入人心的OSI七层模型。

- 众所周知，TCP 承诺可靠、有序、无差错的数据传输。然而，有一天你突然发现服务器组中的某两台之间的基于 TCP 长连接突然变得极慢。你调试了半天，排除了所有的应用程序导致的元素，你终于把视线移到了 TCP 本身。

  这时候你被迫去学习有关 TCP *实现细节*的知识，你了解到网络中其它未部署端到端拥塞控制算法的协议（比如 UDP）很可能会严重影响 TCP 协议的工作（这些协议被称作是 TCP-unfriendly 的），你啃了啃几篇论文，终于了解到工作在 FCFS（*First Come First Serve*，先到者先处理）状态下的路由器处理 *TCP – 无拥塞控制算法协议* 的混合网络有各种各样的问题，典型的比如 *unfairness*，还有 *congestion collapse*——后者的效果甚至会导致在网络的送达率*没有任何提升*的同时仍然（毫无意义地）大大降低 TCP 的工作效率。你最终认识到在路由器上部署 WRR（*weighted round-robin*，带权重的轮询选择）算法或许是一个解决方案。

  这时候，TCP 作为一个*层*，它原本承诺的减轻程序员学习负担这一效果完全没有体现：网络的异常逼迫我们去学习有关 TCP *实现细节*的知识，这些知识**完全没有**包括在 *TCP 提供可靠、无差错的数据传输* 这一契约中。

这样的例子还有很多。比如，遍历二维数组时*按行遍历*和*按列遍历*之间的性能差异甚至意味着程序员最终需要打破语言虚拟机、甚至是 CPU缓存层级结构 的抽象。

软件业的至理名言 “没有银弹” 相信每位读者都听过，而分层作为一种抽象机制同样如此：它承诺的*既能提高效率又能减轻负担*的 “银弹般” 的作用，是不成立的。抽象带来的效率提升，是以更大的学习负担（对实现细节的学习）为代价的：**随着我们创造越来越多、越来越高层的抽象，编程实现会变得更加简单，但成为专业程序员也越来越难。**

看看如今的 Kubernetes 和云原生吧！到底需要强记多少有用没用的知识点、看多少篇晦涩难懂的论文、掌握从网络到操作系统从缓存架构到计组原理的多少方面，才能足以理解这么一个如此高层的抽象，如此庞杂的系统？

**我们已经深陷于不断泄露的实现细节中。成为专业程序员需要了解的知识越来越多，成为有竞争力的专业程序员越来越难——我们会不会看到最终的那个场景，到某一天，泄露的实现细节的总和多到甚至超出了一个个体人类所能掌握的知识量？我们应该做些什么来保持*层*的严密？到底怎样才能使现代软件架构更易理解和掌握？**

或许，随着知识大厦毫不留情地一层层向上堆筑，新一轮软件危机的丧钟已经鸣响。

## 有关*高阶*

相比上一节强调物理或逻辑上客观存在的 “界限”，本节主要强调一种心智活动。这样的心智活动具有一个明显的特征，那就是它具有一种 “递归” 的结构。我们首先从一个概念中抽象出它的形成过程，再将这样的思维过程应用于这个概念*自身*。如此，我们得到的新概念常常具有这样的结构： **`A` 的 `A`** 。

个人认为，本节讨论的思想或许是本文的所有思想中**最为重要的那一个**。这是因为*高阶*这一思想具有很好的方法论层面上的指导意义，并且在实践中操作起来又有趣又直截了当——同时，个人的看法是通过这一思想生成的很多概念对启迪编程初学者的思维具有非常重要的意义（事实上，如果要从*整个技术界*中选一个本科教育一般不大重视、但又极度有价值的术语，我会选*闭包和高阶函数*）。

通过这样一种 “递归式” 的抽象过程得到的 *`A` 的 `A`* 这一形式的概念，我们通常将其称作 **“高阶 `A`”**。比如，将函数代表的过程（`A` -> `B`）抽象出来，再应用于函数*自身*，我们就得到了 *函数的函数*，即 **高阶函数 higher-order function**。个人认为高阶函数和闭包是计算机领域最为重要的抽象之一。

比如，将类型的形成过程（类型描述一个语言中最小单位的类别）抽象出来，应用于类型*自身*，我们就得到了 *类型的类型*，即 *型别 kind* 。

最后，作为压轴，讲一个咱昨晚在一篇屯了半年一年的文章中看到的一个概念。我想这个概念应该是超出大部分读者的经验的，而我觉得它还蛮有意思，所以放到最后来大概说说。我们需要从故事的最开始讲起——可能要过很久才能最终看到这个词。

### “二村映射 Futamura projections”

我们知道大体上编译技术可以分为两种，分别是 **编译器 compiler** 和 **解释器 interpreter**。编译器执行的是一个**两阶段**的过程：首先，自*源代码*编译出*目标程序 target program*，再向*目标程序*提供*输入*，然后得到程序的*输出*；解释器执行的则是一个**一阶段**的过程：解释器接收*源代码*和*输入*，并直接得到程序的*输出*。在本节中，我们主要关注*解释器*。

绝大部分的编译原理教科书和课程都只涵盖这两者。然而，事实上还存在着**第三种**编译技术：某种程度上，它介于解释器和编译器之间，但它*既不是*解释器*又不是*编译器——咱在和同学试讲解这个概念的时候就忘了特别强调这一点。为了方便理解，我先岔开点讲。

稍有函数式经验的读者应该都听过 *柯里化 curried*。如果有一个接收 $n$ 个**输入**的函数，我们可以把它转化成一叠嵌套的*高阶函数*，这叠高阶函数中的第 $i$ 个接收原函数中的第 $i$ 个输入（仅一个），然后返回接收剩下 $ n - i $ 个输入的函数（故为 “高阶函数”）：比如，一个接收 3 个输入的函数（`(A, B, C) -> Output`）经柯里化后会成为一叠高阶函数，这个高阶函数中的第一个接收一个输入 `A`，然后返回一个接收输入 `B` 的高阶函数；再向其提供输入 `C`，才能得到最终结果，即柯里化后的结果是 `A -> (B -> (C -> Output))`。

尽管在支持闭包和高阶函数的语言中柯里化和 *反柯里化 uncurring* 都能以非常浅显（trivial）的方式实现，但仅有少数的编程语言才*一等地*支持了柯里化，比如 ML系语言（Haskell, SML, OCaml, F#）和 Scala。

对于一个经柯里化后的函数，如果我们只向它提供前 $i\space\space(i<n)$ 个*输入*，那么我们必然得不到最终的*输出*，而只能得到一个高阶函数，它会再处理余下的输入：这称作 **部分应用 partial applied**。

讲了这么一大堆，其实主要就是为了*部分应用*这个概念。你可能会想知道为什么我在上文不用*参数*这个惯用说法而是用*输入*，你可能也已经猜到了，这是因为我希望在此强调将解释器理解成一个函数的思想：它的输入是*源代码*和*这段代码的输入*，它的输出是*这段代码的输出*（事实上，很多语言都提供一些十分类似解释器的机制，它们*确实*是函数（比如 `eval`））。

是的，之所以要将解释器理解成一个函数，是因为我正是要借用*部分应用*的思想：既然解释器的输入是*源代码*和*输入*，那么我能不能*部分应用*这个 “函数”（即解释器），首先给定*源代码*，得到一个 “处理剩下参数的高阶函数”（即一个中间程序），然后对于这个中间程序，再提供它的输入，然后得到结果？

**答案是，这是可行的。**事实上，这一过程称作 **部分求值 partial evaluate**，而执行这一过程的**程序**我们称作 *部分求值器 partial evaluator*。部分求值器作用于一个程序和它的一些参数，输出一个该程序对于这组参数 “特化” 后的新的程序。这里的 “一个程序” 我们称作 *源程序 source program*，“新的程序” 我们称作 *残差程序 residual program*。

回到上边有关解释器举例上来。我们将一个解释器和一个源程序传入*部分求值器*中，得到的*残差程序*就是一个接收 *输入* 返回 *输出* 的中间程序。

是不是觉得非常的熟悉……？第一阶段我们提供源代码，得到中间程序；第二阶段我们提供输入，随后得到输出。这是在做什么…？

是的，这就是**编译**。神奇之处在于，我们并没有*手工编写*一个编译器，我们只是将一个解释器和需要被求值的源代码扔给了部分求值器而已——某种意义上来说，我们是 “免费” 得到了一个编译器。

你可能在思考这有什么用？你或许已经将这一思路和*语言虚拟机*联系了起来：众所周知，实现编译器通常比实现解释器**难得多**，那我们能不能弄一个本质其实是一个部分求值器的语言虚拟机，在这个虚拟机上实现一门新的语言只需要编写一个解释器就行了，而语言虚拟机会调用它的部分求值器自动帮我生成对应的、更加高效的 “编译器” 呢？

显然，这是完全可行的——事实上这是有产业实践的。PyPy 是一个 Python 语言的解释器实现，它正是基于以上语言虚拟机的思路实现的：在 PyPy VM 上工作的语言有 Python, Ruby 和 逻辑编程语言 Prolog 等；除此以外，泛语言高级语言虚拟机（HLLVM）GraalVM 的一个重要组件同样如此，工作在 GraalVM 上的语言包括 JavaScript, Ruby, Python, Java, R 等。

这其实就是 **第一类二村映射 first Futamura projection**。

既然是放在 “高阶” 这一主题下的，我们当然要用到这一思想：正如前几段的粗体所提示的，部分求值器其实也是一个**程序**，它自身也能够被部分求值。

回想一开始，`解释器` 的输入是 *源代码* 和 *输入*，我们将 `解释器`（主体）和 *源代码*（主体的其中一个输入）单独拎出来，扔进了部分求值器中，这是第一类二村映射；即是说，在第一类二村映射中，`部分求值器` 的输入是 *解释器* 和 *源代码*。

我们将这一过程——即 “将 *主体* 和 *主体的其中一个输入* 单独拎出来扔进部分求值器中”—— “高阶化”，作用于自身，简单的代换就能得到，我们需要将 `部分求值器`（主体）和 *解释器* （主体的其中一个输入）扔进一个部分求值器中。现在我们需要弄清楚，这个 “高阶过程” 得到的*残差程序*是什么。

注意到，在第一类二村映射中输入只有 *解释器* 和 *源代码*。既然我们已经*部分求值*了前者，那么下一步自然只剩下后者了：向上一段提到的 *残差程序* 提供 *源代码*，我们就能得到 *目标程序*。

向*一个程序*提供源代码得到另一个程序…… 听起来有点像是某个我们已经知道的东西会做的事情。是的，这就是**编译**。我们的这个 “高阶过程” 得到的残差程序其实是一个**编译器**，而我们的这个 “高阶过程” 所做的，正是在**生成编译器**，即，这一过程其实是**编译器的编译器**——如此，我们终于见到了本节概述中提到的 *`A` 的 `A`* 这样的结构，如果你愿意的话，或许你还可以将这称作*高阶编译器*。

这其实就是 **第二类二村映射 second Futamura projection**。

那么，这个 “高阶化” 的过程还能继续下去吗？显然如此。再继续下去，我们就能得到*编译器的编译器的编译器*，也即**第三类二村映射 third Futamura projection**。再往后呢？这点就留给读者自行阅读和思考啦。

## 参考文献

**本节提出的参考文献是本文中各种举例的来源。如果你想深入了解某一个方向或某一点例证，这些参考文献是最好最好的材料——我认为，它们无论是在哪个方面都要远远高于本文粗略而又空泛的简单说明。特别推荐的材料以 ⭐ 标注。**

本节中列出的参考文献是*粗略*按照在文中的顺序排列的。

- Stroustrup, Bjarne. "Evolving a language in and for the real world: C++ 1991-2006." *Proceedings of the third ACM SIGPLAN conference on History of programming languages*. 2007.
- Dan Grossman. Private communication.
- *Hyrum's Law*. https://www.hyrumslaw.com/
- ⭐*The Law of Leaky Abstractions*. Joel Spolsky. https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/
- ⭐*A Philosophy of Software Design* | John Ousterhout | *Talks at Google*. https://youtu.be/bmSAYlu0NcY
- Floyd, Sally, and Kevin Fall. "Promoting the use of end-to-end congestion control in the Internet." *IEEE/ACM Transactions on networking* 7.4 (1999): 458-472.
- Würthinger, Thomas, et al. "One VM to rule them all." *Proceedings of the 2013 ACM international symposium on New ideas, new paradigms, and reflections on programming & software*. 2013.
- ⭐*RubyConf 2013 - Compilers for Free by Tom Stuart*. https://youtu.be/n_k6O50Nd-4
- Perugini, Saverio & Williams, Brandon. (2017). Revisiting the Futamura Projections: A Diagrammatic Approach. Theoretical and Applied Informatics. 28. 15-32. 10.20904/284015. 
- lyzh 聚聚. Private communication.
- Robert Glück. 2009. Is there a fourth Futamura projection? In <i>Proceedings of the 2009 ACM SIGPLAN workshop on Partial evaluation and program manipulation</i> (<i>PEPM '09</i>). Association for Computing Machinery, New York, NY, USA, 51–60. DOI:https://doi.org/10.1145/1480945.1480954
