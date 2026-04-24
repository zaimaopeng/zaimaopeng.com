---
title: "一个真实的负结果，怎样改变了我对 AI memory optimization 的判断"
date: 2026-04-22T22:56:00+10:00
description: "我做这个 memory optimization 项目，本来是想让 personal-agent retrieval 更可信，结果最有价值的收获反而是一个暴露真实瓶颈的负结果。"
tags: ["AI memory", "negative results", "automation", "research workflow", "evaluation"]
categories: ["AI Systems", "Research"]
showTableOfContents: true
---

我开始做这个 memory optimization 项目，是因为自己的 personal agent memory 已经到了一个有点危险的阶段。

它能找回一些东西，已经足够有用；但我还不能完全判断，一个看起来变好的结果，到底是真正的 retrieval improvement，还是刚好撞上了某种表层运气。这个项目最初的目的很朴素：修 benchmark，让 evaluator 真正对齐 live runtime，然后看看能不能把我每天实际使用的 memory retrieval 往上推一截。

真正难受的地方在于，measurement 越干净，结果反而越不漂亮。

当 benchmark 和 evaluator 都修到比较可信之后，几类直觉上很合理的 document-side intervention，比我原来预想得更早到达了上限。这个负结果最后成了整个项目里最有价值的东西，因为它逼我承认：真正的瓶颈不只是 wording，而是 memory surface 之间的结构性竞争。

## 我想要的结果，并不是最后得到的结果

这个项目一开始是想提高一个真实 personal-agent memory 环境里的 retrieval quality。

这种任务天然很容易想把它写成一个 win-story。修 benchmark，修 evaluator，调文档，提 recall，然后写成一次成功优化。

其中一部分确实发生了。benchmark repair 很重要，evaluator compatibility 也很重要。有一批问题如果不先修掉，后面所有结果其实都不值得信。

但一旦 benchmark 更诚实了，evaluation path 也终于跟 live runtime 对齐，整个故事就变了。原本很自然的乐观叙事开始塌掉。小范围的 document-side intervention 的确能改善某些信号，但它们没有真正突破更深层的排序问题。canonical knowledge surface 和 episodic notes 之间的竞争，一直在反复出现。

刚开始这会让人有点失望，但前提是你仍然坚持只把“有没有赢”当作项目价值的主要衡量标准。

## 负结果真正有价值，是因为它澄清了机制

关键不在于某一套策略没有带来很大的 gain。

关键在于，几轮不同 intervention 跑下来之后，可疑的解释空间被不断缩小了。

这让项目从“可能还要继续微调”变成了一件更有信息量的事：在这个环境里，document-side optimization 的上限来得比预想早，真正的瓶颈并不只是 wording，而是 retrieval competition 的结构问题。

这其实比一个浅层 gain curve 更有价值。

一个负结果如果只是说“这个没用”，那它很弱。
但如果它能说明“哪一类东西大概率已经不是主要瓶颈了”，它就开始变得很有分量。

## Benchmark hygiene 不是杂活，它本身就是结果的一部分

这个项目之所以后来能讲清楚，有一个很关键的前提，就是 benchmark 本身必须先修。

一部分看起来像 retrieval failure 的问题，后来发现根本不是 retrieval failure，而是 benchmark-definition failure。有些 expected path 已经过时了，有些 target 已经不存在了，甚至 evaluation stack 本身也还在假设已经不成立的接口。

这意味着，前期真正的问题不只是模型质量，而是 measurement validity。

我觉得这件事不只对这一个项目成立。很多 AI workflow 讨论里，benchmark cleanup 都被当成边角活。但实际上，它经常会改变整个结果面的含义。

## “到顶了”本身也是有价值的信息

当 obvious repair 都做完之后，剩下的 plateau 才真正变成了消息本身。

这部分是我现在最感兴趣的地方。项目不再只是 cleanup，而是开始变成一个 systems lesson：有一些很直觉的优化方向，根本不够强，推不动真正的 bottleneck。

而这种信息，我宁可早点知道。

它节省时间，改变策略，也能防止人一直在错误层面做 endless micro-tuning。

## 我现在更相信什么

我觉得 AI system builder 经常低估负结果，因为它们在叙事上不够漂亮。

但如果 evaluation 是诚实的，系统是真实的，intervention 之间也确实有机制差异，那一个负结果，往往比一个小成功故事更有决策价值。

一个边界清楚、机制更清楚的失败，不是死路。
它更像一张地图。

而在这个项目里，这张地图的价值，其实比硬把 document-side tweak 讲成即将解决问题的方案要高得多。
