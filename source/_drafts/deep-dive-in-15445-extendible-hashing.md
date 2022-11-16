---
title: "深入 CMU 15/445：测试驱动的可扩展哈希表 (Project 2 Extedible Hashing)"
date: 2022-05-25 20:24:11
toc: true
tags:
 - 存储和分布式系统
categories: programming
---

> 本文是一篇*小作品*。

在完成 [MIT 6.824 分布式系统](https://github.com/Ray-Eldath/MIT6.824)课程和 [PingCAP Talent Plan 分布式事务（Percolator 协议）实验](https://github.com/Ray-Eldath/distributed-txn-Ray-Eldath)后，转眼就来到了五月。夏日带着她的晴空和酷暑一步步走来，但先要有声势浩大的狂风和雷雨，方可扣响盛夏之门：每年的这个时节，阴云总在城市的上空盘旋梭巡，所到之处万物都战栗，显出惶惶不可终日的瑟缩模样，被这冷峻又淡漠的天光无情地漂白。而到了每日的黄昏，又会出现一种可爱、神秘的淡蓝色光泽，就连棱角分明的道路和建筑都变得柔和而迷人，仿佛是在补偿日间阴雨的凶险和惨淡。

每当这一年中最为热情的季节到来，就意味着学年的结束。下学期再回到这已经生活了三年的校园，就是最后一年~~，留给中国队的时间不多了~~。

在这一成不变、连日与日间的界限也渐趋模糊的生活中，没有太多东西能够标记不断奔流的时光。

<article class="message is-info">
  <div class="message-header">
    <p><i class="fas fa-image"></i> &nbsp; 关于头图</p>
  </div>
  <div class="message-body">
    头图来自 <a href="https://twitter.com/werlosk" target="_blank">Twitter@Werlosk</a>。
  </div>
</article>

<!-- more -->

CMU 15/445 是一门非常著名的数据库系统公开课。与 [MIT 6.824](http://nil.csail.mit.edu/6.824/2022/schedule.html)、[PingCAP TinyKV](https://github.com/tidb-incubator/tinykv) 等分布式课程不同，该课程关注的是**传统 ACID 数据库系统**的设计与实现，分布式并不作为重点在课程及实验中讲授。当然，即便是 TiDB、CockroachDB、Google Spanner 等 NewSQL 数据库（我将 NewSQL 理解从存储层开始就基于一致性算法、从头设计的全新架构的存储服务，与一众基于 “coordination service” 和一些特设机制进行选主和数据同步（replication）的系统区分开来）[Andy 2016]，其架构和执行层与传统 ACID 数据库系统仍有很强关联。这方面我也有一些初步的想法，但奈何目前知识还不够深入，待我再读一些文章、积累一些产业经验后再好好写写...

老实说，在着手完成这个实验之前总觉得 extendible hashing 应该不会很困难，甚至觉得一定比 [2020 年的 B+ 树实验](https://15445.courses.cs.cmu.edu/fall2020/project2/)简单许多（2021 年 Project 2 从 B+ Tree 换成了 Extendible Hasing）。然而在开始动手之后才发现这个实验丝毫不比 B+ 树简单... 调试的过程十分痛苦。与 MIT 6.824 和 MIT 6.S081 均不同，CMU 15/445 的测试集是**不公开**的，仅能通过一个代码评分平台 Gradescope 获得自己的得分。

**本文提出了一套行之有效的本地测试集，对 extendible hashing 的扩展（grow）和收缩（shrink）的*内部行为*和*外部行为*进行了系统的测试**，这套测试不仅考虑了哈希表*本身*的外部行为，同时还考虑了其与缓存池（buffer pool）的相互作用。

实现 extendible hashing 的另一点主要困难在于，**extendible hashing 并没有一个足够清楚、足够细致的 “纸面” 算法步骤可供参考。**而 B+ 树的主要算法（分裂（split）和合并（merge）等）都在课程教材中以伪代码的形式严格给出了。

本文给出了一个可能的 extendible hashing “纸面” 算法描述。该实现通过了上述本地测试集和 Gradescope 上的全部测试，但从 Leaderboard 排名和用时（逾 60 秒）看来仍有相当的优化空间。我并未仔细思考对该纸面算法的优化，如果你有相关的想法，非常希望你能在本文评论区告诉我 ：）

当然，尽管 B+ 树有更细致、更清楚的纸面算法描述，但在并发问题上，相对于 extendible hashing 只需一把大锁挂到底而言，B+ 树的并发问题则要困难无数倍。lock crabbing 和相关的乐观策略、sibling pointer 和双向 sibling pointer、sequential scan 带来的死锁问题等各种并发场景之间盘根错节带来的复杂性令人瞠目结舌。人类对于本已不堪重负的多核 CPU 又催促又鞭笞，不榨干那仅剩的最后一点并行性就誓不罢休，对于更加复杂的数据结构（如 LSM 树、LSM-WiscKey 变种等），它们的并发实现又会是什么样子，实在是难以想象。~~现在就敢并行化 B+ 树，以后想并行化什么我都不敢想了！~~

> 不过有空了还是想再写写 2020 年的 Project 2 并发 B+ 树...

# 写在前面

在正式开始描述算法之前，先有几点提醒，这些都是我在痛苦的调试中发现极有助益的：

- 在本实验和此后的所有实验中，都**不能**直接申请和释放内存，而必须通过缓存池以页为单位来进行。这意味着每一个方法的每一条返回路径都必须维持两个不变式（invariant）：**其一，方法申请的所有锁都必须被释放**，并且锁的类型要匹配（获取是的读锁释放的也必须是读锁，以此类推）；**其二，pin 的每一个页面都必须 unpin。**注意创建页 `NewPage` 和获取页 `FetchPage` **都会**导致该页被 pin，请确保通过这些操作获得的页最终都被 unpin 释放。

  > 所以 C++ 啥时候加 defer...（

- 对每一个从缓存池中创建或获取的页，**使用 `assert` 确保它们不是空指针 `nullptr`**。例如：

  ```c++
  auto bucket = FetchBucketPage(bucket_page_id);
  assert(bucket);
  ```

  如果泄漏了页（即 pin 的页面没有被 unpin），随着输入规模的增大，最终缓存池会开始返回空指针 `nullptr`，通过使用 `assert` 断言则可以尽早发现这一问题。

# bucket page 和 directory page

# Extendible Hashing

extendible hashing 是一种动态哈希（dynamic hashing）数据结构。bucket hashing（bucket hashing 是一个非常流行的哈希表实现方案，Java 的 `HashMap` 和 C++ 的 `unordered_map` 都是它）等静态哈希（static hashing）在对撞率过高之后需要

无论哪种实现，哈希表暴露的接口都非常简单：`Get` 用于获取与某一个键相对应的所有值，`Put` / `Insert` 用于插入键值对，而 `Remove` 则用于从表中删除某个给定的键值对。对于修改操作，`Insert` 可能会触发扩展，而 `Remove` 则可能会触发收缩。

## Insert 和 Split

`Insert` 操作首先求键对应的 bucket，然后调用其上的 `Insert` 方法。如上所述，`bucket->Insert` 在该 bucket 已满*或*插入重复的键值对时会返回 `false`，由此我们可以得到 split 的条件：当 `bucket->Insert` 返回 `false` **且 `bucket->IsFull` 也返回 `false` 时**，执行 split。

split 的情形又分为两种：

1. **local split：**当 bucket 的 `local_depth` 小于 `global_depth` 时，说明当前键对应的

## 测试集

```c++
template <typename KeyType = uint64_t, typename ValueType = uint64_t>
// NOLINTNEXTLINE
void TestHashTable(const std::vector<int64_t> &keys,
                   const std::initializer_list<const std::initializer_list<int64_t>> &expected,
                   DiskManager *disk_manager, int pool_size = 50) {
  auto hash = HashFunction<int64_t>();
  auto *bpm = new BufferPoolManagerInstance(pool_size, disk_manager);
  ExtendibleHashTable<int64_t, int64_t, Int64Comparator> ht("blah", bpm, Int64Comparator(), hash);
  for (size_t i = 0; i < keys.size(); i++) {
    auto key = keys[i];
    if (keys.size() < 1000) {
      cout << key << "=" << std::bitset<8>(hash.GetHash(key)) << " ";
    }
    // test insert
    ASSERT_TRUE(ht.Insert(nullptr, key, key));

    // test receive current
    std::vector<int64_t> res;
    ASSERT_TRUE(ht.GetValue(nullptr, key, &res));
    ASSERT_EQ(res[0], key);
    if (keys.size() < 1000) {
      cout << "pass receive " << res[0] << endl;
    }
    // test receive all
    if (keys.size() < 1000) {
      for (size_t j = 0; j <= i; j++) {
        std::vector<int64_t> res2;
        auto key2 = keys[j];
        ASSERT_TRUE(ht.GetValue(nullptr, key2, &res2))
            << "Failed to get all. key: " << key2 << " (" << std::bitset<8>(hash.GetHash(key2)) << ")";
        ASSERT_EQ(res2[0], key2) << "Failed to receive all at [0, " << i << "], key: " << key2;
      }
      cout << "pass receive all [0, " << i << "]" << endl;
    }
  }
  ht.VerifyIntegrity();

  auto directory = reinterpret_cast<HashTableDirectoryPage *>(bpm->FetchPage(0));
  directory->VerifyIntegrity();
  // expectation check
  if (expected.size() > 0) {
    cout << endl << "=== CHECKING EXPECTATION: ";
    ASSERT_EQ(1 << directory->GetGlobalDepth(), expected.size());
    uint32_t idx = 0;
    for (auto expected_bucket : expected) {
      auto bucket_page_id = directory->GetBucketPageId(idx++);
      auto bucket =
          reinterpret_cast<HashTableBucketPage<int64_t, int64_t, Int64Comparator> *>(bpm->FetchPage(bucket_page_id));

      uint32_t bucket_size = 0;
      std::vector<int64_t> bucket_val;
      for (uint32_t i = 0; i < BUCKET_ARRAY_SIZE; i++) {
        if (bucket->IsReadable(i)) {
          bucket_size++;
          bucket_val.emplace_back(bucket->KeyAt(i));
        }
      }

      ASSERT_EQ(expected_bucket.size(), bucket_size);
      for (uint64_t val : expected_bucket) {
        cout << val << " ";
        ASSERT_EQ(std::count(bucket_val.begin(), bucket_val.end(), val), 1);
      }
      bpm->UnpinPage(bucket_page_id, false);
    }
    cout << "===" << endl << endl;
  } else {
    LOG_DEBUG("skip buckets expectation check");
  }

  // test Remove
  auto max_key = *std::max_element(keys.begin(), keys.end()) + 1;
  for (const auto &impossible_key : GenerateKeys(max_key, max_key + 100)) {
    ASSERT_FALSE(ht.Remove(nullptr, impossible_key, impossible_key));
  }
  std::vector reversed_keys{keys};
  std::reverse(reversed_keys.begin(), reversed_keys.end());
  for (const auto key : reversed_keys) {
    ASSERT_TRUE(ht.Remove(nullptr, key, key))
        << "Failed to Remove key " << key << " (" << std::bitset<8>(hash.GetHash(key)) << ")" << endl;
  }
  ASSERT_EQ(directory->GetGlobalDepth(), 0);

  bpm->UnpinPage(0, false);
  delete bpm;
}
```



# 结语

## 参考文献

- **[Andy 2016]**: Pavlo, Andrew, and Matthew Aslett. "What's really new with NewSQL?." *ACM Sigmod Record* 45.2 (2016): 45-55.

## 另请参阅

对于 Extendible Hashing 的实现，尚缺乏值得参考的资料。以下列出的内容我**并未仔细研读**，在此仅作有限的补充。

- 
