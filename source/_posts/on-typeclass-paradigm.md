---
title: "从 SML 到 Scala：简单考察 typeclass 范式的演变和各种实现，以及🎉🎉🎉"
date: 2021-02-13 17:35:11
cover: /img/on-typeclass-paradigm.jpg
thumbnail: /img/on-typeclass-paradigm.jpg
toc: true
tags:
 - 程序设计范式（paradigm）
 - 程序设计语言
 - Rust
categories: programming
---

> 本文是一篇「小作品」
>
> ~~这次要是还写巨长我就吃了渚薰（（（（大雾）~~

Typeclass 范式是对于 *表达式问题 Expression Problem* 的一个重要的解。在我了解的编程语言范式中，个人认为，typeclass 范式是*较为优雅*的一个。本文将简要考察这一范式本身，以及更加重要的：它在各种编程语言中到底如何落地。具体而言，本文将在各种落地语言中构造**同一个**示例：一个类似 Ruby 中的 Comparable *mixin*，或者 Java 中的 Comparable 接口，并且演示这些结构如何对既有的类型同样具备可扩展性。

阅读本文需要一定的代码基础，尤其是对 typeclass 范式的认识和相关的编码经验。本文并不会对文中的举例作详尽解释。

<!-- more -->

## 表达式问题 Expression Problem

先来简单说说*表达式问题*。表达式问题是编程语言设计中的一个重要问题，它**非常直接**地与我们日常的编程工作联系起来——这也是其之所以重要的一大原因。这一问题最早是由 Philip Wadler（是的是的，就是那位 Mr.λ wwwwww，不知道他的这点轶事的朋友可以去油管上看看他的 talk [Propositions as Types](https://youtu.be/aeRVdYN6fE8?t=3090)（话说我最早是看 Facebook 的一个介绍 Haskell 在他们内部大规模生产级落地的 talk 知道这位 Mr.λ 的，[那个 talk](https://youtu.be/mlTO510zO78) 也相当不错，大家也可以去看看~））在讨论 Generic Java（好像是 Oracle 给 Java 泛型这块设计的项目取的代号）的设计时提出的。表达式问题提出的背景是使用一门编程语言提供的表达能力来表达一个表达式系统（`Expr`），这个系统需要具有可扩展能力（这正是名字中 *表达式* 的来源）。简单来说，表达式问题考察：**一个语言如何支持*扩展*一个既有的类型（`datatype`），常见的操作是，向这个类型增添*子类型（`case`）*，或者向所有的类型增添*操作（通常以方法的形式）*。**

将这个统领表达式的类型（`Expr`），按所有的*子类型（case）*和所有的*操作（operations）*两个维度分别列全，就可以得到一张表，这张表格的行即*子类型*列即*操作*。广义来讲，这张表格很好地表明了*函数式编程 Functional Programming, FP* 和 *面向对象编程 Object-Oriented Programming, OOP* 之间的**完全对立**甚至是**正交**——一个偏重行（即子类型）的可扩展性，一个偏重列（即操作，通常是*方法*或*函数*）的可扩展性（当然，前提是你要同意，Java 不代表 OOP，而 FP 和范畴论、Monad、Functor 等各种有用没用的概念并没有什么关系）。

> 这可能是本文（或者是本博客？）所有文字中*最重要的一句废话*：
>
> **Java 不代表——至少不完全代表——OOP，而 FP 和范畴论、Monad、Functor 等各种有用没用的概念其实并没有什么关系。**

具体见文末 “主要引述来源”。

## Typeclass in SML: Module System

ML 族语言自 1970 年代，LCF 定理证明器（theorem prover）的元语言（**M**eta **L**anguage）演化而来。SML（Standard ML）以及 ML 族语言，作为 *严格求值（strict evaluation）*、*静态类型*的*函数式*语言的典例，启发了大量语言的设计，并对编程语言的形式化定义和验证等研究方向做出了重要贡献。

SML 的 *模块系统 Module System* 为语言细节的隐藏提供了强大的工具（SML 的*模块*其实很有意思，比较值得学习研究），它或许是历史上第一个提出（并实现）这一范式的编程语言：

{% codeblock typeclass.sml lang:sml %}
signature COMPARABLE = sig
  type elem
  val compare: elem -> elem -> int
end

functor Ord (X: COMPARABLE) : sig
  val le: X.elem -> X.elem -> bool
  val eq: X.elem -> X.elem -> bool
  val ge: X.elem -> X.elem -> bool
end = struct
  fun le x y = X.compare x y <= 0
  fun eq x y = X.compare x y = 0
  fun ge x y = X.compare x y >= 0
end

structure IntComparable : COMPARABLE = struct
  type elem = int

  fun compare x y =
    case Int.compare (x, y) of
         LESS => ~1
       | EQUAL => 0
       | GREATER => 1
end
structure IntOrd = Ord(IntComparable)
{% endcodeblock %}

`signature` / `structure` 之间的配合是 *ML Module System* 的重要方面，前者基于类型签名约定了一个*接口*（同样，不是 Java 意义上的接口…… 每次都要在术语处特别和 Java 划清界限实在是 😒😒😒），而后者则是对这一接口的实现。有趣的是，和绝大多数语言中类似*接口*的机制的设计不同，这一实现过程并不是简单的要求 “一模一样”，而是存在着复杂有趣的 ***签名匹配 signature matching*** 机制。这一机制提供了接口中重要的*隐藏*方面。

回归正题，我们首先定义 `signature COMPARABLE`，这类似于 Haskell 中的 `class`。随后，我们希望为内置类型 `int` 实现这一 typeclass（这一过程称作 *ascription*）——我们定义了 `structure IntComparable : COMPARABLE`（这类似 Haskell 中的 `instance`），没错，冒号 `:` 正是 “实现自” 的意思。

至此我们已经完成常规意义下的 typeclass 范式——定义一个*接口*，并使既有类型实现这个接口。

出于演示目的，我复杂化了这个示例：除了单纯的 typeclass 范式之外，此处演示了基于 typeclass 范式的后半截逻辑：在一个既有类型的 typeclass 实现之上，定义 “任何实现了这个 typeclass 的类型，都具有这些操作”。这是通过一个看起来有点奇怪的关键字 `functor` 实现的。`functor` （称作 *module function*）基于一个已有的 `signature` 完成这一转换路径：如果一组元素 `{A}` 在一个类型 `X` 上有定义，那么 `X` 上就会有另一组元素 `{B}`（在这里，`{A} = { type elem, fun compare }`；`X = int`；`{B} = { fun le, fun eq, fun ge }`）。

> 这个转换路径听起来是不是很像 mixin…？
>
> 以及这个 “后半截” 逻辑听起来是不是很像 Rust 中的 `From` / `Into`…？

在这段代码中，我们首先通过 `IntComparable` 实现了 typeclass 范式，随后 `functor Ord` 在 `IntComparable` 上的实例化（这个语法也很有意思，`Ord(IntComparable)` 这种结构的语法其实正是在提示这种 “参数传递” 的 “实例化” 意味）完成了后半段：我们使用 typeclass 范式为一个业已存在的类型 `int` 实现了一组操作，随后基于这组操作衍生出了一组新的操作。

这一示例的演示如下：

{% codeblock >folded output from *Standard ML of New Jersey* REPL v110.98.1 lang:sml %}
- use "typeclass.sml";
[opening typeclass.sml]
signature COMPARABLE = sig
  type elem
  val compare : elem -> elem -> int
end
functor Ord(X: sig
  type elem
  val compare : elem -> elem -> int
end) :
sig
  val le : X.elem -> X.elem -> bool
  val eq : X.elem -> X.elem -> bool
  val ge : X.elem -> X.elem -> bool
end
structure IntComparable : COMPARABLE
structure IntOrd :
  sig
  val le : X.elem -> X.elem -> bool
  val eq : X.elem -> X.elem -> bool
  val ge : X.elem -> X.elem -> bool
end

val it = () : unit

- IntComparable.compare 42 42;
val it = 0 : int

- IntOrd.eq 42 42;
val it = true : bool

- IntComparable.compare 2 4;
val it = ~1 : int

- IntOrd.le 2 4;
val it = true : bool
{% endcodeblock %}

一点局限性在于，我们需要为 typeclass 的两个部分赋予不同的名字：`IntComparable` 和 `IntOrd`。重名在 SML 中式不允许的——后出现的定义将会 *掩蔽 shadow* 掉先出现的定义。

*ML Module System* 是一个完备、丰富、强大的模块系统，它的能力远远不止于此（比如，*sharing constraints* 等并未提及）。

最后，顺带一提，我们在一个接口中包含了一个*类型*（`type elem`），并且接口中的其它定义依赖于这个类型定义（`val compare : elem -> elem -> int`），这在很多语言中被称作 **联合类型 associated type**。

---

由于 SML 相当冷门（因而更难以熟悉），我们在这一节上花费了大量笔墨。接下来看一看其它语言中的 typeclass。

## Typeclass in Haskell: `class` & `instance`

Haskell 是一门*静态类型*的*函数式*语言，这门语言的最大特点，在于它是 *惰性求值 lazy evaluation* 的——这是一个不同于绝大多数语言的设计决策。除此以外，高度拥抱~~犯愁~~范畴论及相关术语（而不是如 Scala 语言设计者一般在这个问题上相当谨慎，引自 Dean Wampler），同样是这门语言的重要特征（然而正如它的核心发明者之一 SPJ Simon Peyton Jones 说的那样，Haskell 没有学习 F# 采用更加保守的命名而是全面拥抱理论数学，或许是一个相当错误的决定，引文见文末）。

typeclass 这一范式正是由 Haskell “定义”，一般认为 Haskell 语言是这一范式的起源：

{% codeblock typeclass.hs lang:haskell %}
class Comparable a where
    comp :: a -> a -> Integer

instance Comparable Integer where
    comp x y = 
        case compare x y of
          GT -> 1
          EQ -> 0
          LT -> -1

le :: Comparable a => a -> a -> Bool
le x y = comp x y <= 0

eq x y = comp x y == 0

ge x y = comp x y >= 0
{% endcodeblock %}

大部分部分都是不言自明的，在此不做过多解释。显然，得益于强大的类型推导机制和精心设计的语法，Haskell 中实现 typeclass 的代码量是相当小的。

我们仅为 `le` 标注了类型：类型声明是可选的（虽然建议标出），因为 Haskell 可以帮你推断出来。由于在可变性上采取了更加严格（因而更加*函数式*）的规定，Haskell 不必像上一节提到的 SML 一样在类型系统上 “开洞”，引入所谓的 *value restriction* 和 *dummy type*。

> 我要~~用腐朽的声音~~喊出：**[Hoogle](https://hoogle.haskell.org/) 天下第一！！😋**
>
> **强烈建议其它所有良好支持 typeclass 范式的语言都要（至少是对语言标准库）有这么一个 `class` 的查询引擎，避免重复轮子……**

演示如下：

{% codeblock >folded "output from *ghci* lts-15.9" lang:plain %}
Prelude> :load typeclass.hs
[1 of 1] Compiling Main             ( typeclass.hs, interpreted )
Ok, one module loaded.
*Main> comp 42 42
0
*Main> eq 42 42
True
*Main> comp 2 4
-1
*Main> le 2 4
True
{% endcodeblock %}

同样基本是不言自明的。

## Typeclass in Rust: trait

Rust 是主要由 Mozilla 开发、现依托于开源社区和 [Rust Foundation](https://foundation.rust-lang.org/) 独立运行的*面向*函数式、静态类型的非托管语言，它直接编译到机器码，并通过精心设计的 *所有权 ownership* 机制达成了非托管语言难以做到的*内存安全*（具体可以看看咱博客的 [Rust 系列文章](https://ray-eldath.me/tags/Rust/) 😉）。

Rust 中采用了 `trait` / `impl` 原语实现这一机制。个人**浅见**是 `struct` 基本等同 `record`，`trait` 基本等同 `class`，而 `impl` 基本等同 `instance`，所以说 Rust 的表达力基本没有太多超出 ML 系语言的一般水平。

与上两例不同，Rust 并没有使用 *柯里化 curring* 为函数传参的传统，亦未为这一特性提供一等支持：

{% codeblock typeclass.rs lang:rust %}
use std::cmp::Ordering;

trait Comparable {
    type Elem;
    
    fn compare(&self, y: Self::Elem) -> i8;
}

impl Comparable for i32 {
    type Elem = i32;
    
    fn compare(&self, y: Self::Elem) -> i8 {
        match self.cmp(&y) {
            Ordering::Less => -1,
            Ordering::Equal => 0,
            Ordering::Greater => 1,
        }
    }
}

fn le<TT, T: Comparable<Elem = TT>>(x: T, y: TT) -> bool { x.compare(y) <= 0 }
fn eq<TT, T: Comparable<Elem = TT>>(x: T, y: TT) -> bool { x.compare(y) == 0 }
fn ge<TT, T: Comparable<Elem = TT>>(x: T, y: TT) -> bool { x.compare(y) >= 0 }
{% endcodeblock %}

作为一门函数式气氛较弱的语言，强制的显式类型标注（并重复两次）、以及需要通过 *泛型 generic* 指明类型约束使这段 Rust 代码稍显冗杂——大多数工业级语言都只能做到这个程度。

演示如下：

{% codeblock >folded "output from *Rust Playground*" lang:rust https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=07c98bcedef5ca99b40e4988e09d8137 ">  Try this on your own 😉" %}
fn main() {
    println!("{}", 42.compare(42));
    println!("{}", eq(42, 42));
    println!("{}", 2.compare(4));
    println!("{}", le(2, 4));
}

------------------ Standard Error ------------------
   Compiling playground v0.0.1 (/playground)
warning: function is never used: `ge`
  --> src/main.rs:23:4
   |
23 | fn ge<TT, T: Comparable<Elem = TT>>(x: T, y: TT) -> bool { x.compare(y) >= 0 }
   |    ^^
   |
   = note: `#[warn(dead_code)]` on by default

warning: 1 warning emitted

    Finished dev [unoptimized + debuginfo] target(s) in 0.77s
     Running `target/debug/playground`
------------------ Standard Output ------------------
0
true
-1
true
{% endcodeblock %}

有关 Rust 的更多内容，欢迎访问 [Rust Language](https://www.rust-lang.org/zh-CN/learn) 以及查看本博客中 [其它有关 Rust 的文章](https://ray-eldath.me/tags/Rust/)。

> Try this on your own 😉: https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=07c98bcedef5ca99b40e4988e09d8137

## Typeclass in C++ 20 Indian Concept: A Failed Endeavour

你可能会疑惑，**啥啥啥？？🤨🤨 C++ 啥时候也有 typeclass 了？？？😯😯😯**

然而，正如本节的标题所提示的：这是一次*失败的努力*。语言设计提案最终未能获得 *共识 consensus*。

来看看这个失败的提案（被称作 **Indiana Concept**）：

{% codeblock typeclass_failed.cpp lang:cpp %}
concept Comparable<typename T> {
    int operator<=>(const T& x, const T& y);
}

concept_map Comparable<int> {
    int operator<=>(const int& x, const int& y) {
        if (x >= y) return 1;
        else if (x == y) return 0;
        else if (x <= y) return -1;
    }
}

template<Comparable T> bool le(const T& x, const T& y) { return (x <=> y) <= 0; }

template<Comparable T> bool eq(const T& x, const T& y) { return (x <=> y) == 0; }

template<Comparable T> bool le(const T& x, const T& y) { return (x <=> y) <= 0; }
{% endcodeblock %}

其实也蛮简洁的嘛。

为什么没有获得通过？typeclass 它难道不比模板、还有 *TMP 模板元编程 这种诡异至极的奇葩* 香多了吗？

这一提案最初于 2006 年正式提出，并最终在 2009 年决定正式从即将发布的语言规范草案中移除。Bjarne 在 *HOPL3*（详见文末 “主要引述来源”）中无不惋惜地说：

> That decision cost us three years of **hard work and much controversy** … We could not agree to "fix" concepts to make them usable by most programmers and also ship the standard (more or less) on time. Thus, "concepts" – **the result of years of work by many competent people** – was **removed** from the draft standard.

个人认为，typeclass 范式所倡导的 *be explicit* 和 C++ 本身一贯执行的 *be implicit* 哲学之间的~~阻抗失配~~不契合是该提案失败的重要原因（我的[另一篇博文](https://ray-eldath.me/programming/thoughts-on-rust-1/#Rust-%E2%80%9Cbe-explicit%E2%80%9D)中对这点有更深入的解读）。*HOPL3* 中列举的原因有：

- 各个层面的语言设计进展相当缓慢，关键问题仍未解决。难以达成*共识*。
- 一些精心编写的 `concept_map` 和 `late_check` 会导致类型系统 *不完备（unsoundness）*。
- 现有的这一部分规范极度复杂，长达 91 页。可读性很差。
- 在标准库中落地 concept 的工作量令人意想不到地巨大无比。
- 性能**极低。** *（despite "heroic efforts from Doug Gregor"）*启用了 concept 的编译器编译速度比未启用的编译器慢**不止十倍**。主要的 C++ 编译器供应方表示，只要有超过 20% 的性能损耗，他们就不会同意提案。

后来由 Bjarne 主导设计的新提案取得了长足进展，形成了在 GCC 6 中得以实验性发布的 Concepts TS，并经少量修改最终形成了我们现在见到的 C++ 20 Concepts：concept 被定义为**类型的谓词**（即 `constexpr <type> -> bool`。

> 个人看来这是一个相当精妙的设计思路。很好地利用了 C++ 现有的基础设施：对于 `constexpr` 的支持。
>
> 就和 `auto` 很好地利用了*模板类型推导*这一基础设施一样。

## Typeclass in Scala 2: implicit

“Scala 是 Scala 是一门编译到 JVM 字节码的多范式语言。在所有工业级编程语言中，Scala 以其惊人的复杂度和优雅程度而著称。” Scala 中的 typeclass 范式主要是通过 `trait` 和 *隐式 implicit* 实现的。

就和 SML 的 *Module System* **远远不止** typeclass 一样——Scala 的 *implicit* 同样如此。Scala 的 *implicit* 是这门语言最为强大的特性之一，除了 typeclass，它还能够表达诸如 *隐式参数 implicit parameter*、*隐式证据 implicit evidence*、*类型限定 type constraint*、*扩展函数 extenstion* ，等等等等，好用有趣的语言范式上。

> 因为*隐式*的用法实在是太多太混，于是 Scala 3 就把这一个关键字上承担的过多功能分拆到了几个不同的关键字上…

所以，这就是*隐式*的其中一种用法：

{% codeblock typeclass.sc lang:scala %}
trait MyComparable[T, TY] {
  def compare(x: T, y: TY): Int
}

object MyComparable {
  implicit val intIntComparable = new MyComparable[Int, Int] {
    override def compare(x: Int, y: Int) = x.compare(y)
  }

  def le[T, TY](x: T, y: TY)(implicit instance: MyComparable[T, TY]) =
    instance.compare(x, y) <= 0
  def eq[T, TY](x: T, y: TY)(implicit instance: MyComparable[T, TY]) =
    instance.compare(x, y) == 0
  def ge[T, TY](x: T, y: TY)(implicit instance: MyComparable[T, TY]) =
    instance.compare(x, y) >= 0
}
{% endcodeblock %}

由于*隐式*本身用法多样带来的复杂性，上边这段 Scala 代码看起来比较复杂。再来看一下演示：

{% codeblock >folded "output from *Scastie*" lang:plain https://scastie.scala-lang.org/YxSlxJPnRXKYn9Lh5dEVig ">  Try this on your own 😉" %}
// Exiting paste mode, now interpreting.

trait MyComparable
object MyComparable

scala> implicitly[MyComparable[Int, Int]].compare(42, 42)
val res0: Int = 0

scala> MyComparable.eq(42, 42)
val res1: Boolean = true

scala> implicitly[MyComparable[Int, Int]].compare(2, 4)
val res2: Int = -1

scala> MyComparable.le(2, 4)
val res3: Boolean = true
{% endcodeblock %}

在代码和演示中，我们使用了 *隐式参数*（`implicit instance`）和一个标准库中的对象 `implicitly` 查找当前上下文中符合类型要求的隐式。这一过程称作 *召唤 summon*（**是不是很中二啊www 😉**）。

> Try this on your own 😉: https://scastie.scala-lang.org/YxSlxJPnRXKYn9Lh5dEVig

## 结语，及主要引述来源

为与各个编程语言的惯用法和文化相适应（更加*地道*），各节中的例子均有一些*实现细节*层面的修订。这使它们看起来并不完全一样（有一些通过*柯里化*传参，一些是直接传参；一些方法名为 `compare`，一些是 `comp` （为了避免命名空间冲突），一些使用的是 *太空船运算符 spaceship operator* `<=>`（这个名字真是太可爱了www 😙））。

> 顺带一提，这篇文章里的大部分演示，其实都是我看着 Haskell 的那段代码对着写的…
>
> ~~Snipaste 天下第一！~~

**主要引述来源：**

- *The Expression Problem*, Philip Wadler.  http://homepages.inf.ed.ac.uk/wadler/papers/expression/expression.txt
- Griesemer, R., Hu, R., Kokke, W., Lange, J., Taylor, I. L., Toninho, B., ... & Yoshida, N. (2020). Featherweight go. *Proceedings of the ACM on Programming Languages*, *4*(OOPSLA), 1-29.
- MacQueen, D., Harper, R., & Reppy, J. (2020). The history of Standard ML. *Proceedings of the ACM on Programming Languages*, *4*(HOPL), 1-100.
- *Notes on SML97's Value Restriction*, Geoffrey Smith.  http://users.cs.fiu.edu/~smithg/cop4555/valrestr.html
- *Simon Peyton-Jones: Escape from the ivory tower: the Haskell journey*, SPJ  Simon Peyton-Jones.  https://youtu.be/re96UgMk6GQ
- 《简单聊聊编程语言的哲学，以及关于 Rust 的一些想法 (1)》, Myself.  https://ray-eldath.me/programming/thoughts-on-rust-1
- **HOPL3:** Stroustrup, B. (2020). Thriving in a crowded and changing world: C++ 2006–2020. *Proceedings of the ACM on Programming Languages*, *4*(HOPL), 1-168.
- *No 'Concepts' in C++0x*, Bjarne Stroustrup.  https://accu.org/journals/overload/17/92/stroustrup_1576/
- https://en.cppreference.com/w/cpp/compiler_support
- https://github.com/Ray-Eldath/whatever/blob/master/main/src/main/scala/cats/monad/Monad.sc
- 一些语言的 Playground**（都超级好用！）**：
  - Rust：https://play.rust-lang.org/
  - Scala：https://scastie.scala-lang.org/
- Haskell 部分使用 *ghci* REPL 编写
- SML 部分使用 *Standard ML of New Jersey* REPL 编写

---

> 两天啥也没干狂肝两篇博文，可真是把我榨干了。
>
> 这个假期应该不会出新博文了… 想静下心来休息下，学点东西啥的。

最后，yet again：**各位新年愉快！**



——以及，今天是我的18岁生日，祝我自己成年快乐。🎂🎂🎂🙌🙌🙌🎈🎈🎈🎉🎉🎉

希望今后能学到更多有趣的东西，创造一些更有价值的事物，去到更加遥远的地方，了解更加广阔的世界，认识更多有趣的人——并和他们一起前行。

各位，**两周后见！**👋👋👋

*<全文完>*



> 今天网易云推的歌怎么都那么好听啊
>
> www