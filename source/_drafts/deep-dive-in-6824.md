---
title: "深入 MIT 6.824：实现 LeaseRead 和全异步 shardkv"
date: 2022-03-14 18:22:40
toc: true
tags:
 - 存储和分布式系统
categories: programming
---

「生活，就是当你忙着做其它计划时，发生在你身上的事」：要问起开始学 MIT6.824 的缘由，这是一句恰如其分的描述。一个学期结束，原本的计划是深入研究一下编程语言和形式化验证，然后换到一个和操作系统关系更大的岗位；不知不觉却变成了写个数据库，在做操作系统相关的工作前先试试存储的方向——刚放假时我还对自己说，我是绝对不会喜欢上存储的。也许人们对自己还没下过苦功的事情，就是提不起劲的吧？

于是春节假期开始，明显一年比一年淡漠的气氛（今年甚至没看拜年祭）环绕，倒也是为思考提供了较好的场所：没有多少亲戚前来拜访，也就没有多少生硬蹩脚的玩笑和紧张尴尬的时刻需要消化。到现在，整一个月，算是把 MIT6.824 彻底完成了。

老实说，这并不如我想象中的难。在上手之前总觉得 824 的高不可攀，非顶尖学府的高手不可；但自己*突然*完成之后却发现虽然过程并不是「出乎意料地」顺利，但产出确实是出乎意料地好。

[我已开源了](https://github.com/Ray-Eldath/MIT6.824)这份实现。这一实现稳定通过了 Lab 1 到 Lab 4 的每一个测试点（至少 1000 次，通常 2000 次；其中 `Linearizability2B` 通过了 10000 次——这么做是因为是在提出 pocourine 的文章中，作者说 “他*没有找到任何一个*线性一致性测试不能发现的错误”），完成了 Lab 4 的两个 Challenges（即标题中的*全异步 shardkv*），同时还*额外*实现了 LeaseRead with nop 的优化，使得读请求无需经共识层。参考资料中列出的一份实现（似乎）同样了 LeaseRead，但却未能实现 nop 而是采用了有些蹩脚的轮询方法——本文中提出的解决方案或许是对如何在 MIT 6.824 中实现这一优化的较好参考。另一方面，我谨慎地组织了代码结构，保证代码的粒度适宜——既不引入过多重复，又不引入过多函数（是不是想起了洗试管的原则？嘿嘿），每一次修改都执行了充分的回归测试，并且非常谨慎地操作 squash 和 cherry-pick，保证修改最少、最必要，且和我**完成 Lab 的进度**严格对应：这意味着读者可以从代码的提交历史中明确看到*某个 Lab 和下一个 Lab* 之间应要做哪些修改。

<!-- more -->

> 你可以从以 `-test` 结尾的测试分支中看到是**多么令人沮丧的 debug 过程**被隐藏在了 cp & squash 后的表象之下：
>
> ![Revert "Revert "这次一定能行.jpg""](/img/deep-dive-6824/image-20220309113933597.png)

总体 Lab 的难度大概是 Lab 4B Challenges > Lab 4B > Lab 2 (2C = 2B > 2A > 2D) >> Lab 4A = Lab 3 = Lab 1。

本文将分两个部分，拆解实现中两个较为新颖的侧面：一个是（正如我刚刚提到的）全异步的 shardkv（即 Lab 4 Challenges），另一个是如何实现 LeaseRead with nop 使得读请求无需过共识层（若有机会，本博客兴许会更新完整的实现笔记，而不仅仅是两个较为新颖的侧面）。**为不影响各位读者的学习体验，本文将尽可能以「问题情形——解决方法」的形式组织**，当你发现无法通过的测试时，你可以阅读一节「问题情形」，在继续阅读「解决方法」*之前*，你应自行排查自己的实现是否足以排除了这些情形，随后你可以对照相比本文中提出的解决方法而言，你的解决方法有什么不同。

最后，显然的一点是，没有分布式和存储领域的前辈们慷慨无私地分享自己完成这一课程的宝贵经验，这篇文章不可能诞生。特别需要感谢 [OneSizeFitsQuorum 的一系列讲解](https://github.com/OneSizeFitsQuorum/MIT6.824-2021)，尽管本文使用的方案与之迥异，却也殊途同归。其它对本文作者很有帮助的参考资料亦附在文末，推荐各位读者前往阅读。**此外，本文作者对本文的所有疏漏负全部责任。若有任何问题、讨论（比如，你的解决方法很可能比我的更好，我*非常希望*你能在评论区告诉我）或建议，欢迎在评论区留言。**

# 全异步 shardkv

Lab 4B 无疑是难度最大的。相比有示例结构和明确参考的 Lab 2 raft，Lab 4 要求大家从零开始设计分片的 KV 数据库，**从头设计**必须实现的分片移交（shard handoff）机制，没有任何示例和文字描述可做参考，还要保证这一机制不会违背线性一致性。

Lab 4 附有两个 challenges，分别要求我们实现失效数据的清除（Challenge 1）和异步变更配置的 shardkv（Challenge 2）。首先需要强调的一点是，如果你希望实现异步变更配置的 shardkv（也就是完成两个 challenges），**一定要从设计的最初始就考虑这样做。**

**我们先来思考一下 challenge 2 到底是什么意思。**Lab 4 中要求我们实现一个*分片*的分布式键值数据库，其核心是当配置变更（也就是从切片（shard）到组（replica group）的映射关系发生变化时）时集群要同步反映这些变更：原先负责这个切片的组应不再负责这个切片，而新被分配这个切片的组应开始服务，并且要在和原组相同的数据上继续。在切片移交（handoff）的过程中不能接受客户端请求。

一个非常显然的实现就是**同步**的实现：我们编写一个新的常驻 goroutine，这个协程定期更新配置并计算变更（diff），如果发现变更，停止**所有的**客户端请求和**整个** applier，移交切片并更新配置，待所有切片都就绪后才恢复客户端请求和 applier 循环。

**这种简单的做法无疑效率低下。原因有二：**首先，一个组会负责多个切片，而配置变更时需要移交和同步的切片通常只是一个组负责的多个切片的一部分。举一例：在配置变更 `{1 [100 102 101 101 102 100]} -> {2 [100 100 100 101 101 101]} ` 中，组100负责从负责切片 `[0 5]` 变更至负责切片 `[0 1 2]`，其中只需要同步 `[5 1 2]`，而切片0在整个过程中都可以不中断服务（上述同步实现显然不能做到这点）；其次，一次变更时可能需要来自不同组的多个切片，而一个切片可以在收到移交的数据时就立即开始服务，无需等待别的移交。还是上面的例子，变更配置时组100需要来自组102的切片1和组101的切片2，一旦收到来自组102的切片1，这一切片就能立刻开始服务。

**所以，要完成 challenge 2，我们需要将配置更新、分片移交和分片恢复服务三个阶段完全异步化。**当检测到配置更新时，仅仅停止需要同步的切片，不能阻塞客户端请求和 applier 循环；此后移交切片，并且在收到对应切片时立即恢复对该切片的服务。

已有的众多实现（甚至连测试都暗示了... shardkv 的 `TestConcurrent2` 测试点上边的注释写的是 `fetch shard contents`...）都采用了 “拉切片” 的同步方法。即当检测到配置变更时，由新被分配某个切片的组向它的「前辈」去 “要” 切片，相关的文章通常宣称这样的做法会更简单，尤其是在需要从宕机（crash）中恢复的若干场景下。

本文质疑这种说法。**本实现证明了使用 “推切片” 的同步方法亦是可行的，两者间并无显著的复杂度区别，无论是就 RPC 次数还是宕机恢复逻辑的复杂程度而言。**本节余下部分谈论的「问题场景」对 “拉切片” 依然适用，且要在那种同步方法中解决所有这些问题很可能需要相似的机制。鉴于两者并无显著的复杂度差异，各位读者可任意地自由选择同步方法。

## 总体结构

需要强调的是，shardkv with challenges 的要点是江**配置更新**、**分片移交**和**分片恢复服务**这三个部分完全解耦并使之异步化。任何实现方案都要做到这些，而此处给出的方案只是若干可行解中的一种。

在 Lab 4A 中我们已经实现了负责提供一致 `Config` 的 shardctler，而 Lab 4B 的 shardkv 泽通过 `shardctler.Clerk` 与其定期通信并拉取配置。注意到从配置更新到分片恢复服务之间有若干阶段，每个阶段间都有可能发生宕机和换主，请确保你的实现能够处理**各种情形（在下文「问题场景」和「解决方法」将更多地涉及具体的错误场景）**，这意味着每个节点要对当前配置和「更新步骤进行到哪个阶段」有*一致*的认识。

因此，整个 reconfiguration 过程的大部分逻辑应在 applier 循环中实现。对于一轮 reconfiguration，集群异步地按三个步骤工作，并依赖 Raft 层实现状态的一致转移：

1. **更新配置：**首先 Leader 会定期拉取配置。当发现配置的 `Num` 比本地配置更大**且所有分片的状态都是正常状态时**，调用 Raft 层 `Start`。对应示意图中 `DoUpdateConfig`。
2. **更新配置：**接下来由包括 Leader 在内的每个节点的 applier 循环接收自 Raft 层推送的最新配置，由这个最新配置*更新当前配置*、*计算 diff* 并据此*更新每个分片的状态*（这个接下来还会讲到），同时*开始分片移交（handoff）*。对应示意图中 `applyConfig`（这是一个由 applier 循环根据消息类型调用的方法）。
3. **移交分片：**接收到移交（handoff）消息的组的 Leader 调用 `Start`，成功后向原组发送移交成功消息。对应图中 `Handoff`。
4. **移交分片：**每一个节点的 applier 循环接收移交消息并对应更新自己的 kv map。对应图中第一个 `applyMsg`。
5. **分片恢复服务：**当原组的 Leader 收到分片移交成功消息后，调用 Raft 层的 `Start`。对应图中 `HandoffDone`。
6. **分片恢复服务：**包括 Leader 在内的每个节点的 applier 循环接收自 Raft 层推送的配置更新完成消息，清理对应的 key（Lab 4 Challenge 1）并将这些切片恢复到正常状态，为下一轮 reconfiguration 做准备。对应图中第二个 `applyMsg`。

示意图如下：（我知道我的字很丑，请别喷我... 呜呜呜呜 🥺🥺🥺）

![](/img/deep-dive-6824/pic1.jpg)

注意到在一次 reconfiguration 的流程全部完成之前不会更新下一个配置，因为在最后一步恢复分片状态前一定有分片的状态不是正常状态，而当有分片是非正常状态时步骤1不会启动。

可以看到整个 reconfiguration 过程还是比较复杂的，并且有若干细节问题需要仔细考虑，这会在下文中详述。由于本节仅对总体设计做概观式的鸟瞰，以下将再简要讨论三个问题：

### 问题1：分片状态

第一个问题是，上文不断提到分片除了 `Config` 中给出的 `shard -> gid` 对应关系外，还有一个**额外的**「状态」需要维护。分片有哪些状态？应该如何管理这些状态？

当一个组的节点接收到 Raft 层推送的最新配置时，要计算 diff 并更新分片状态（第2步），对于一轮 reconfiguration，切片移交的发送方和接收方都会这么做，从而产生两种可能：一个分片从别处移给我，一个分片从我这交给别处。算上正常状态，一共有三种状态：`Serving`、`Pulling` 和 `Pushing`。注意到在这三种状态下都是**不能提供服务**的。

另外是如何管理这些状态。一种方案是约定特殊的 `Config.Shards` 值，比如 `-1` 表示 `Pushing`、`-2` 表示 `Pulling`，这样做可能会给 diff 算法引入额外的复杂度，并且更新 `Config.Num`（是当所有特殊值都消除时才更新 `Num` 吗？）的策略还会引入别的问题。更好的方案是在 `Config` 之外维护一个额外的 `ShardState` 数组，并通过 Golang 的常量枚举语法提升代码的可读性：

{% codeblock shardkv/server.go lang:go %}
type ShardKV struct {
    mu           sync.Mutex
    me           int
    rf           *raft.Raft
    mck          *shardctrler.Clerk
    config       Config
    lastConfig   Config
    groups       map[int][]string
    applyCh      chan raft.ApplyMsg
    make_end     func(string) *labrpc.ClientEnd
    gid          int
    ctrlers      []*labrpc.ClientEnd
    maxraftstate int // snapshot if log grows this big
    cid          int32
    serversLen   int

    shardStates [shardctrler.NShards]ShardState
    kv          map[string]string
    dedup       map[int32]int64
    num         int64
    handoffCh   chan Handoff
    done        map[int]Done
    doneMu      sync.Mutex
    lastApplied int
}

type ShardState int

const (
    Serving ShardState = iota
    Pulling
    Pushing
)
{% endcodeblock %}

### 问题2：RPC 消息类型和 RPC handler 的个数

在上文的步骤描述中，出现了**两个** RPC handler 和**两对共四种** RPC 消息：用于移交分片的 `HandoffArgs` 和 `HandoffReply`，以及用于完成分片移交的 `HandoffDoneArgs` 和 `HandoffDoneReply`。

你可能会觉得，一个 RPC handler 可以既处理发出去的请求又处理回来的回复，所以只用一个 RPC handler 和一对 RPC 消息 `func Handoff(args *HandoffArgs, reply *HandoffReply)` 就能解决问题。**这样做行不通。**具体原因请见下文「问题场景」一节。

### 问题3：持久化

最后一个问题是哪些状态应该被持久化。如上所述，从配置更新到分片恢复服务之间有若干阶段，每个阶段间*都有可能*发生宕机和换主，所以请务必仔细考虑有关持久化的问题，在本部分的剩余两节会更加详细地讨论之。

---

对 reconfiguration 的整个流程的鸟瞰式概括就到此结束了。如果你还没有开始动手完成 Lab 4B，**为达到最好的学习效果，你应关掉本文，开始自行编写实现**，并仅当遇到难以解决的问题时（确保你已经足够努力地 debug）继续阅读本文的剩余部分。

## 问题场景

在**设计并**实现（主要是*设计*）上面的流程中遇到了若干问题，其中有一些问题的解决方法并没有详细包含在上述流程中，主要是因为显然由读者自行发现这些问题并自行思考解决的机制将会十分有益。本节将总结我在设计和过程中遭遇的若干会引起问题的场景，需要注意这并不一定覆盖了所有可能的问题场景，而仅仅是我遇到的。在本文的余下部分，称一次移交的发送方为 “源组”，接收方为 “目标组”。

第一类是与分片移交机制本身相关的场景，共有四种：

![](/img/deep-dive-6824/pic2.jpg)

- **a. handoff lost update**

  组100发起移交，随后该节点宕机（Z字型线表示节点宕机）。接收组完成了更新并开始服务请求，而源组的新 Leader 又发起了一次移交，接收组应用已经应用过的移交，造成*状态回退*。

- **b. lost handoff done**

  组100发起移交，随后该节点宕机。接收组完成更新并返回移交成功消息，由于源组的原发送节点已经宕机，导致它无法向 Raft 层提交移交完成的消息，造成~~思索~~*死锁*。

- **c. lost handoff**

  组100的原 Leader S0 通过*步骤1*和*步骤2*在组内更新完配置，正准备发起移交时宕机了，新选出来的 Leader S1 以为原 Leader 已经发送了移交，于是一直等待移交完成消息，造成死锁。

- **d. lost config**

  组100的 Leader S0 完成了*步骤1*，而在*步骤2*前恰好宕机，而在 S0 恢复后一开始不是 Leader，从 Raft 层传来的 `Config` 不会触发移交，从而丢失了一次 reconfiguration。

还有一类场景是与消息去重相关的，但这与去重机制的具体实现相关，在此不再赘述。

## 解决方法

> 本节包含大量 “剧透”，可能影响你的学习体验。

### 解决问题场景

- **a. handoff lost update**

  或许简单的去重就能解决这个问题，然而这必须在同一个组的每个节点都使用相同的 `ClientId` 时才行得通。我并没有使用这种方法（或许这样做会更简单，欢迎尝试了这种方法的读者在评论区指出其优劣）。我的解决方法是在应用状态前增加两下检查：首先检查在移交的分片对应本地状态中，是不是有 `Pulling` 状态的分片，如果没有则明显说明这是一个重复的移交。然而这样做并不足够，因为在目标组收到重复移交之前可能已经进行了下一轮 reconfiguration，所以还需要检查 `HandoffArgs.Num` 是否是最新的。*仔细思考*对于比较 `HandoffArgs.Num` 和 `currentConfig.Num` 出现的各种情况，什么时候应该返回 OK 什么时候应该返回 timeout。 

- **b. lost handoff done**

  **这就是在*问题2*中提到的不能使用一对 RPC 消息和一个 RPC handler 就搞定移交的原因。**这里的问题本质上是当一个节点需要对某个 RPC 的**回复**达成共识时，回复可能被网络延迟太久以至于到达时该节点已不再是 Leader，无法调用 `Start`。注意到**所有依赖 RPC 回复的 `Start` 调用都会出现这个问题，你的实现可能以*别的形式*引入了它。**设计复杂的 RPC 协议并不能解决这个问题而只是延后了问题。解决方法其实很简单，上文也已提到：使用**多一对 RPC 消息和多一个 RPC handler** 来传递「完成操作」的信息，并且信息的发送方要**轮询**接收方直至成功发送（「完成操作」消息的接收方也就是「操作」消息的发送方）。注意，在配置更新后，「操作」消息发送方的地址可能会被丢失。

- **c. lost handoff**

  该场景有显然的解决方法，在此概不赘述。

- **d. lost config**

  这是一个有点意思的场景，应该有多种解决方法。我的方法是在*步骤1*传入 Raft 层的 `shardctler.Config` 之外额外包一层，一并传入「是哪个节点提交的这个 `shardctler.Config`」，当收到 Raft 层推送的配置时，如果这个值和本节点的编号匹配，则**无视本节点可能不是 Leader**，立即发起移交：

  {% codeblock shardkv/server.go lang:go %}
  type Config struct {
      Conf      shardctrler.Config
      Committer int
  }
  
  func (kv *ShardKV) DoUpdateConfig() {
  updateConfig:
      for {
          time.Sleep(UpdateConfigInterval)
          if !kv.isLeader() { continue }
          for _, state := range kv.shardStates {
              if state != Serving {
                  kv.mu.Unlock()
                  continue updateConfig
              }
          }
          num := kv.config.Conf.Num + 1
          kv.mu.Unlock()
          kv.rf.Start(Config{kv.mck.Query(num), kv.me})
      }
  }
  
  func (kv *ShardKV) applyConfig(latest Config, commandIndex int) {
      if commandIndex <= kv.lastApplied {
          kv.Debug("reject Config due to <= Index. lastApplied=%d latest=%+v", kv.lastApplied, latest)
          return
      }
      // ...
      kv.updateShardStates()
  }
  
  func (kv *ShardKV) updateShardStates() {
      latest := kv.config
      kv.Debug("applying Config  states=%v", kv.shardStates)
      handoff := make(map[int][]int) // gid -> shards
      for shard, gid := range kv.lastConfig.Conf.Shards {
          // ...
      }
  
      if kv.isLeader() || kv.me == latest.Committer {
          kv.handoff(handoff, latest.Conf, kv.copyDedup())
      }
  }
  {% endcodeblock %}

### 去重

去重和如何移交去重表会带来很多问题，**但是只要你使用了合适的去重方案，这一切都会变得十分简单。**我推荐的方案是使用 `SequeceNum` 对消息进行去重（客户端的每个请求都要带 `ClientId` 和**递增的** `SequenceNum`，且仅在服务器返回 OK 时递增 `SequenceNum`），这会使得避免各种去重相关的问题变得非常容易。我一开始是用随机而非递增的 `RequestId` 进行去重，导致用于消息去重的数据结构比较复杂，而且遇到的问题也很多，移交去重表的流程也很麻烦。

另外一个有关去重的细微问题是如果使用 `SequenceNum` 机制去重请求，那么客户端**必须只有一个 goroutine 会访问 `SequenceNum` 这个状态。**如果是 Clerk 这很容易做到，但是在移交分片时就可能需要额外的考虑。我使用了一个 `handoffCh` 通道和的一个**常驻的 goroutine 协程**来保证这点，这应该还算是一个不错的结构：

{% codeblock shardkv/server.go lang:go %}
func (kv *ShardKV) DoPollHandoff() {
    for handoff := range kv.handoffCh {
        handoff.args.Args = Args{ClientId: kv.cid, SequenceNum: kv.num}
    nextHandoff:
        for {
            for _, si := range handoff.servers {
                var reply HandoffReply
                ok := kv.sendHandoff(si, &handoff.args, &reply)
                if ok && reply.Err == OK {
                    kv.num++
                    break nextHandoff
                }
                if ok && reply.Err == ErrWrongGroup { panic("handoff reply.Err == ErrWrongGroup") }
            }
            time.Sleep(UpdateConfigPollInterval)
        }
    }
}
{% endcodeblock %}

> 思考一下 `handoffCh` 应该是 buffered 还是 unbuffered channel？如果是 buffered channel，应该有多大？

## 代码结构



