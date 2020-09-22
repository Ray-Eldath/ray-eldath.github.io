---
title: "First Step Towards FPGA (1): SystemVerilog Quick Take & Pros and Cons"
date: 2020-06-16 13:40:51
cover: /img/fpga-1.jpg
thumbnail: /img/fpga-1.jpg
language: en
toc: true
tags:
 - 硬件编程
 - SystemVerilog
 - FPGA
categories: hardware
---

Every time programmable hardware programming is mentioned, Verilog or SystemVerilog comes to our mind — such fact, IMHO, is ironically contrasts with another interesting, if not consensus, but at least first impression of those hardware newbies just like me, that the fundamental software and development toolkit in hardware programming field is far from diverse, mature and easy-to-use. Comparing to software engineering, there are not too much languages, tools or methodology to let you pick and choose, even among the limited available choices, most of them are either lack some important features, or just too expensive to investigate. Undoubtedly my first step towards FPGA, looking around and pick a combination of *language,  simulator and testing method*, is a brief journey, but it also involved too many investigation as well as unexpected disappointment, which makes this journey more difficult, and more tiring. This article is intended to outline some of my conclusion, which is what I’m using now, and what I have used but quited.

<!-- more -->

For me, there is a long-lasting haunting thought — writing a CPU, and that is the beginning of the story. About half a month ago I set sailed, and till now I have finished a testing library for Verilator, and a very simple MIPSs CPU that have nothing to do with pipeline, trap, and whatnot. So conclusion comes first: I’ve done most of these in combination of **SystemVerilog (language) + [Verilator](https://verilator.org/) (simulator) + [CLion](https://www.jetbrains.com/clion/) (IDE) + [althas](https://github.com/Ray-Eldath/althas) (testing) + Xilinx Vivado (synthesis)**. I will explain the reason just below. I’ll not gonna say this is the best workflow of hardware programming, specifically FPGA programming, it still have many incompetence in terms of my requirements which I’ll talk about them later as well, but I do hope this article could help you find a workflow that just suits yourself.

**This article will only discuss the very first part of my workflow, namely the SystemVerilog part.** Remaining techniques will be discussed in the follow-on articles of this series, so *stay tuned!*

# Verilog / SystemVerilog ?

~~JUST DON’T ask why VHDL is not mentioned.~~ I won’t discuss VHDL here, for one thing I know nothing about this language, and for another I think it’s somewhat considered obsolete or just not recommended in many existing articles like [this one](https://www.alse-fr.com/Do-I-need-SystemVerilog.html).

## What’s different?

*SystemVerilog* has been described as a superset of *Verilog*, which is another predominant language in hardware programming field, and it certainly is. SystemVerilog have an *impressive outstanding compatibility* with Verilog, most synthesis environment, at least Xilinx Vivado, allows you to instantiate Verilog module in SystemVerilog module and vice versa. Many Verilog definition could be directly translated into SystemVerilog, without or with only little altered. Here is a piece of code taken from my  CPU project [astrio](https://github.com/Ray-Eldath/astrio) (currently it's still in private, I'll publish it once I finished pipelining, trap, AXI bus and implementation on Zynq SoC, all sort of things), it can help you — I assume you already knew Verilog, and understand what these `assign`, `always` means — glimpse at the vanilla-SystemVerilog, and understand what’s new there:

{% codeblock >folded pc.sv lang:verilog https://github.com/Ray-Eldath/astrio/blob/master/astrio/pc.sv "see code on GitHub" %}
import Parameters::*;
import PCType::pc_cmd_t;
import Types::addr_t;

module PC(
    input PCType::pc_cmd_t cmd,
    input addr_t load_pc,
    input bit rst,
    input bit clk,
    output addr_t pc,
    output addr_t inc_pc
);
    initial pc = InstStartFrom;

    addr_t next_pc;
    
    assign inc_pc = pc+4;
    always_comb begin
        unique case (cmd)
            PCType::NONE:
                next_pc = pc;
            PCType::INC:
                next_pc = inc_pc;
            PCType::INC_OFFSET:
                next_pc = inc_pc+load_pc;
            PCType::LOAD:
                next_pc = load_pc;
            default:
                next_pc = pc;
        endcase
    
        if (rst == 1)
            next_pc = InstStartFrom;
    end
    
    always_ff @(posedge clk)
        pc <= next_pc;
endmodule : PC
{% endcodeblock %}

It’s not difficult to find out that there are many differences comparing to Verilog. I’ll highlight those are *most important or influential* in terms of the actual programming work:

### `typedef` in SystemVerilog
You may notice that there are some `addr_t` stuff in the code, and even if you are just a newcomer of C, you may still notice that here we adhere to the general naming convention of a `typedef`, namely suffixed whatever a `typedef` with `_t`, which I think is just inherited from C’s naming conventions. So as you may assumed, **SystemVerilog do have `typedef`.**  Let’s look at the definition of `addr_t`:

{% codeblock types.sv lang:verilog https://github.com/Ray-Eldath/astrio/blob/master/astrio/types.sv "see code on GitHub" %}
package Types;
  typedef logic unsigned [31:0] addr_t;

  typedef logic signed [31:0] op_t;
  typedef reg signed [31:0] op_reg_t;
  typedef logic unsigned [4:0] reg_id_t;

  typedef logic unsigned [31:0] inst_t;
  typedef reg unsigned [31:0] inst_reg_t;
endpackage : Types
{% endcodeblock %}

Okay, now it’s pretty clear. Generally `typedef` in SystemVerilog is no different from its counterpart in C/C++. Intuition always right, just use `typedef (type) (name)` to define your own data type in SystemVerilog to make your code more concise and configurable. BTW, `typedef` is a great replacement for *parameter* in many cases, use it wisely, and your code will definitely looks better. **But, wait,** what the hell is the `package`?

### `package` & `import`: Package system and separate compilation
Extract the common definition of your code, wrapped them in a *package* or whatever is always a good idea. It improves the readability of your code, **alleviate namespace conflicts**, and enables the *separate compilation*, i.e. when the code are recompiled, only the modified part is needed to be recompiled while the common and relatively less frequently modified part can be excluded, so less files will be recompiled, therefore speed up the recompilation. SystemVerilog supports such encapsulation by providing keyword `package` and `import`: the former is for encapsulate, the latter is for import it. One thing should be noticed: if you read my codebase that linked hereinbefore, you may find that *only `typedef`, functions and some parameters are encapsulated in package*, this is because typically they are the common part of your code. I won’t encapsulate my *module* in package, seems to me it just don't deserve.

### `enum` & *enhanced* `case` clause: *enumeration* works like a charm
In the `pc.sv` code above, there is a input `cmd` typed `PCType::pc_cmd_t`. You may guess that this is a `typedef` defined in package `PCType`, but what the actual type is? Continual reading, the input `cmd` is passed into a `unique case` clause, perhaps it’s something akin to enumeration? And `unique` means that there are no overlapping between the matching conditions, so only **one** block will be triggered in any cases?

You are right! Here’s the definition of `pc_cmd_t`:

{% codeblock pc_type.sv lang:verilog https://github.com/Ray-Eldath/astrio/blob/master/astrio/pc_type.sv "see code on GitHub" %}
package PCType;
    typedef enum bit [1:0]{
        NONE, INC, INC_OFFSET, LOAD
    } pc_cmd_t /* verilator public */;
endpackage : PCType
{% endcodeblock %}

Here we define a `enum` that occupies a 2 bit wide `bit`, then aliased it as a `pc_cmd_t`, where we adhere to the appreciated naming conventions. The whole definition is encapsulated in a `package` named `PCType` as I discussed before. Still there are something weird — what is the `bit`? And why there is a comment `/* verilator public */`?

I am going to talk about the `bit` thing right now, but for the Verilator thing, till we know what Verilator is and what can it do can we grasp at the idea, so the answer is in the follow-on articles of this series.

### `logic` & `bit`: abundant but systematic built-in datatype
In addition to the native datatypes directly inherited from Verilog, lots of new datatypes are described and added. Presumably the most crucial subset of these novel datatypes (I think many of them is just *excessive* abstraction and design and actually not so required but make the language considerably more complex. I’ll back to this point later) is `logic` and `bit`. Technically you can regard `logic` as `wire` plus `reg`. Datatype `logic` wipe out the delimited `wire` and `reg`, for synthesizer will deduce the underlying hardware (a wire or a register) automatically, using the context. But if you are declaring a *tristate gate* like `inout`, you have to use `wire` but not `logic`. Why is that? The reason is `logic`-typed variables can only have one *driver*.

No surprise here, `logic` still have four state: `0` (low), `1` (high), `X` (unknown) and `Z` (tristate gate high-impedance), but in many cases we can firmly believe  that only two state, `0` and `1`, will be involved. And that’s what `bit` used for. It’s a apt datatype for defining enumeration.

Actually there are plentiful datatypes newly added into SystemVerilog, and most of them are not mentioned here due to some reasons, and I’ll come to this later.

### `always_XXX`: *enhanced* `always` clause enable fine-grained *elaboration hint*
Maybe the most complex, sophisticated and perplexing part of Verilog/SystemVerilog is the `always` clause. Without intensive study, profound understanding and very careful development, unintended *latch* or *flipflop* will be deduced with no warnings, which surely is a bad thing. SystemVerilog partially solve this problem by giving you *fine-grained elaboration hint*. There are three types of new enhanced `always` clause:

- `always_comb`: Next time you write down `always @(*)`, try `always_comb`! This type of `always` clause *family* will automatically listen to all left-side wires and registers in your `begin … end` block, thus *sensitive signal list* should **not** be specified. Simulator and synthesizer will ensure that only *combinational logic* is deduced, and if not, a warning (or error?) will be thrown. You should not use *delayed assignment (`<=`)* in the context as well.
- `always_ff @(...)`, `always_latch @(...)`: Take their names literally, these two clause is for *sequential logic*. One will *hints* simulator and synthesizer to deduce *flip-flop*, the other will deduces *latch*. The sensitive signal list is **required**, and you should specify the sensitive signals in it, exp. `always_ff @(posedge clk)`, just like in Verilog.


## Silver Bullet?

**But** still, SystemVerilog is **not** the silver bullet, and here are some downsides.

### Relatively poor ecosystem

Before you switch to SystemVerilog (which I highly recommend, as many renowned hardware workshops and companies had done so), **the very first as well as the most significant** thing you should deliberately consider is ecosystem. That is to say, **does the synthesizer, implementation program or IDE support SystemVerilog?** And if not, apparently SystemVerilog can not be a practical option. The good news is that almost every conceivable mainstream FPGA manufacture provides development environment with bundled SystemVerilog support — but the bad news is SystemVerilog is only supported in their relatively newer version of software. Take Intel and Xilinx as example, only *Quartus II 11.1 and above* and *Vivado v2017.3 and above* support SystemVerilog. So if you’re planning to program on old devices, such as a Xilinx Spartan-6, you are not able to use SystemVerilog since Vivado only supports 7 generation FPGA products (*Zynq 7000 SoC*, *Artix-7*, *Kintex-7* and *Virtex-7*). Notice that some so-called individual manufacture, in particular [Alchitry](https://alchitry.com/collections/all) — the manufacture of the somewhat relatively reputable *cheap, newcomer-friendly* FPGA development board *Mojo*, do not provide SystemVerilog in their dedicated IDE as well.

![Xillinx Nexys 3 and Mojo v3 — They are all Spartan-6 based.](/img/fpga-1-boards.jpg)

### SystemVerilog is (overly?) complex

As I mentioned before, SystemVerilog is **a complex HDL.** But the problem is it’s not just simply complex, it’s somewhat **overly and unnecessarily** complex. Comparing with Verilog, actually many enhancements are reasonable, adequate and competent, but the other side is there are many features just trying to make coding in SystemVerilog as closer to coding in some high-level language, let’s say C++ and Java, as possible, and this way is just not so great. Many language features and keywords, such as datatypes `int`,  `longint`,  `real`, crappy multi-thread support, keyword `automatic` for recursive functions, they are not used in my day-to-day programming, but makes SystemVerilog unreasonably complex — as a HDL, whose theoretical role is just describes wires between hardware structures. I’ll say that this attempt, trying to "disguise" SystemVerilog as a high-level language will indeed improve the testing experience. (Personally speaking the `TESTBENCH`-based and `$display`-based testing methodology of Verilog is incredibly inefficient and grueling, I suppose the SystemVerilog way of doing things will be a lot easier) But, I still hold the belief that HDL should be a HDL *solely*, and high-level language should be a high-level language solely. It’s just redundant, and usually a bad idea to let one side covers the other side, or to "be" the other side. Given the consensus that hardware validation and testing, especially the generation of testcases should be performed in a high-level language, maybe use a high-level language to do such things will be a great idea — and *[Verilator](https://www.veripool.org/wiki/verilator)* is created for this. I will cover Verilator and related topics in the next article.

After all, this is just a personal preference and personal perspective, and *language is just a tool*. If it’s too complicated, just pick out the acceptable part: and this is exactly what I’m doing. Use what I called vanilla-SystemVerilog is just stick to the principle that SystemVerilog is **nothing but** a HDL language, you should only use the HDL part of it, take is as a "better Verilog", use entirely a handful of features and enhancements including what I mentioned before.

### Limited features

Even if your environment fully support SystemVerilog in elaboration, synthesis and implementation, your switch may still not very pleasant as you may expected. Specifically, my *Vivado v2019.2* satisfactorily supports SystemVerilog in normal develop procedure, but you **can’t** add your SystemVerilog modules to your Block Design directly. Overall this is not a big problem, in my case all I need is just to create a wrapper in Verilog then linked IPs to that wrapper, but your mileage may vary.

![Xilinx Vivado v2019.2](/img/fpga-1-vivado.jpg)

## It all depends.

Eventually comes the golden rule of selecting things in the realm of techniques, technicians, languages and programmers — “**It all depends.**” As I said, *language is nothing but a tool*, how to use it and to what extent will you use it in the daily basis, is just personal predilections. Tons of tutorials and blog posts will teach you how to use SystemVerilog as a high-level language and do validation things in such way elegantly — and I bet many people are good at it. But I’m just not on their side, nor am I appreciate their methodology.

Hope this article will help you get the basic difference, pros and cons of SystemVerilog, comparing with Verilog, and then decide whether to put it in your own workflow. The next passage will cover another facet of my workflow — Verilator, it lets you simulate and validate your DUT in C++/System C, which is a lot better then do such things in HDL, personally speaking.