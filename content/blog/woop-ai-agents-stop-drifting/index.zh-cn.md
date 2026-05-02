---
title: "我用 WOOP 让 AI Agent 停止漂流"
date: 2026-05-02T10:27:21+10:00
lastmod: 2026-05-02T10:27:21+10:00
description: "把 WOOP 从一个人的行动方法改造成 AI Agent 的工作流契约之后，我发现真正有价值的不是多一个计划模板，而是让 agent 能识别自己的漂移、触发恢复动作，并且接受能输掉的评测。"
tags: ["AI", "Agent", "WOOP", "工作流", "OpenClaw"]
categories: ["技术"]
showTableOfContents: true
slug: "woop-ai-agents-stop-drifting"
---

这次改 `Deep Work` 的时候，我意识到一个很有意思的问题：

很多 AI Agent 并不是不知道用户想要什么。

它知道 wish。它能复述你的目标。它甚至能写出一个看起来很合理的 plan。

但它会在执行中漂走。

漂移有很多种样子：Planner 给出一堆平权选项但不做推荐；Research 一直收集资料，不肯停下来合成；Coding Agent 修完一个问题以后没有把验证跑干净；Project Ops 把一个本来可以结项的项目继续假装打开；Deep Work 在长任务中变成反复报告“我接下来会做什么”，而不是继续产生交付物。

这很像人类的“漂流”：你有愿望，但行动触发权被默认选项、情绪、惯性和环境接管了。

所以我试着做了一件事：把 WOOP 从一个人的行动方法，改造成 agent 的工作流契约。

WOOP 本来是 Gabriele Oettingen 推广的心智策略：Wish、Outcome、Obstacle、Plan。它把愿望和结果想清楚，然后逼你看见关键障碍，最后写出一个 If-Then 预案。[^woop]

在人类身上，它大概是：

```text
我想要 X。
我想得到 Y。
我最容易被 Z 卡住。
如果 Z 出现，我就做 A。
```

放到 agent 身上，这个结构变成了：

```text
用户真正要的交付物是什么？
什么证据说明这件事已经成功？
我这个 agent 最可能在哪个内部模式里漂移？
如果这个漂移信号出现，我必须立刻执行什么恢复动作？
```

关键是第三步。

Obstacle 不能写成“外部环境很复杂”。对 agent 来说，真正有用的障碍是 agent-side obstacle：它自己的失败模式。

比如：

- Planner 的障碍不是“选择很多”，而是“用选项堆砌代替推荐”。
- Auto Research 的障碍不是“信息不够”，而是“证据已经足够时还继续收集”。
- Coding Agent 的障碍不是“代码库复杂”，而是“验证失败后仍然软性 closeout”。
- Deep Work 的障碍不是“任务很长”，而是“长任务中反复计划，忘了交付”。

这样 WOOP 就不再是心理学名词，而变成了一个 agent workflow contract。

## 我把它整合到哪里

第一步是 `Deep Work`。

我给它加了一个内部 WOOP loop：在 kickoff、阶段切换、验证失败、最终 closeout 之前，agent 要静默检查 Wish、Outcome、Obstacle、If-Then Plan。

但很快我发现，WOOP 不应该只属于 Deep Work。

它更像一个共享的 alignment method，所以我又把它整合到了四个更容易漂移的 skill：

- `planner`：在 route selection 之前做 WOOP，防止选项堆砌和过度规划。
- `coding-agent`：在多文件或模糊修复任务前做 WOOP，防止扩大 patch 或跳过验证。
- `auto-research`：在 intake 和 stop/continue 决策点做 WOOP，防止无限收集。
- `project-ops`：在开项、转向和结项时做 WOOP，防止 false-open 或过度推广未完成材料。

这次最有意思的改动不是“多写了几段说明”。

而是每个 skill 都开始有一个可执行的恢复句式：

```text
如果我开始做 X 这种漂移动作，那么我必须先做 Y 这个恢复动作。
```

这就是 implementation intention，也就是 If-Then planning。Peter Gollwitzer 这一支研究反复强调的不是“多想想目标”，而是提前把触发情境和行动绑定起来。[^gollwitzer]

对 agent 来说，这等于给工作流加了一个微型中断处理器。

## 评测结果

我做了三层评测。

第一层是人工 scorecard，问的是：改完以后，skill contract 是否更能约束漂移？

结果是：

```text
36.8 -> 47.2 / 50
```

第二层是 deterministic contract coverage harness。它不问“我感觉是不是更好”，而是从 git 里的 baseline instruction surface 和当前 worktree 对比，检查这些 skill 文本里是否真的出现了可执行的 drift guard。

结果是：

```text
48.0 -> 60.0 / 60
relative improvement: 25.0%
```

第三层是 current-thread live replay pack。我设计了 5 个行为 case，分别覆盖 Planner、Coding Agent、Auto Research、Project Ops 和 Deep Work。

这一次单臂 replay 的结果是：

```text
45 / 50 = 90%
```

这里我故意用了“单臂”这个词。

因为它只是证明：当前 WOOP 条件下，这套工作流可以在真实任务里产生 artifact、验证证据和诚实边界。

它还不是 blinded A/B proof。

这件事非常重要。

## 这个分数是不是我自己设计出来让自己好看的？

这是我觉得整篇文章最值得写的部分。

如果一个人改完架构以后，再设计一个评分系统，然后宣布“我的架构分数更高了”，这当然有自证嫌疑。

这个质疑不但合理，而且必须正面放进方法里。

所以我给这套评测补了一个 validity audit。它的第一句话基本上就是：

```text
Did the evaluator design a scoring system after the integration in order to make the new architecture look good?
```

我的 defense 不是“相信我”。

真正的 defense 是让结果能输。

具体做法是：

- baseline 从 git `HEAD` 取，不手写一个很弱的 strawman。
- deterministic harness 和原始输出都留在仓库里，可复跑。
- live replay 的评分不奖励“说出 WOOP 四个词”，而奖励 artifact、validation、scope control、failure recovery 和 limitation honesty。
- 设置 negative controls，例如 safety gate 不应该因为 WOOP 突然全部大涨。
- 写清 falsification criteria：如果 WOOP 让 user-drag 增加、plan-only loop 增加、Research 继续无限收集、Coding 仍然跳过失败验证，那就必须缩小或否定行为改进 claim。
- 承认当前最硬的结论只是 contract-level improvement，不是 production telemetry。

这和 Goodhart's Law 有关系：一旦一个指标变成优化目标，它就可能被系统学会投机。[^goodhart]

所以好的评测不是让你赢。

好的评测要允许你输。

我还把下一步实验写成 blinded A/B replay：同一组 prompt 分别跑 baseline 和 WOOP 条件，把输出随机改名成 Run A / Run B，让评分者在不知道条件的情况下打分，最后再 unblind。

这借鉴了 registered reports 和 blinded interpretation 的思路：先冻结方法，再看结果；先锁定解释，再揭晓条件。[^registered][^blind]

只有到了那一步，才更接近行为层面的证明。

## 真正的收获

这次 WOOP 改造让我更确定一件事：

很多 agent 问题不是“智能不够”，而是“执行中的自我约束不够精确”。

一个 agent 可以理解愿望，却仍然在执行中漂移。

它需要的不是更长的 plan，而是一个能在漂移发生时触发的恢复动作。

WOOP 的价值就在这里。

它把一个抽象愿望编译成一个可执行的 If-Then：

```text
如果我开始偏离 Outcome，
那么我先处理 Obstacle，
再继续执行 Plan。
```

对人来说，这是夺回行动触发权。

对 agent 来说，这是把“知道目标”变成“能在执行中回到目标”。

我现在不想把这篇文章写成“WOOP 已经证明让所有 agent 表现提升 X%”。

那还太早。

更诚实、也更有价值的结论是：

WOOP 让我的 agent workflow contract 变得更可执行了。它把漂移从一种模糊感觉，变成了可以命名、可以触发、可以评测、也可以被证伪的东西。

这已经足够重要。

因为 agent 最危险的状态，往往不是失败。

而是看起来还在努力，其实已经漂走了。

[^woop]: WOOP is commonly described as Wish, Outcome, Obstacle, Plan; see the public WOOP resource at [woopmylife.org](https://woopmylife.org/).
[^gollwitzer]: For implementation intentions and If-Then planning, see the University of Vienna publication page on [mindset theory of action phases and if-then planning](https://ucrisportal.univie.ac.at/en/publications/mindset-theory-of-action-phases-and-if-then-planning/).
[^goodhart]: David Manheim and Scott Garrabrant, ["Categorizing Variants of Goodhart's Law"](https://arxiv.org/abs/1803.04585).
[^registered]: Nature's [Registered Reports guidelines](https://www.nature.com/nature/for-authors/registered-reports) describe pre-registering methods and analyses before the research is conducted.
[^blind]: For blinded interpretation as a bias-control method, see ["Blinded interpretation of study results can feasibly and effectively diminish interpretation bias"](https://www.sciencedirect.com/science/article/pii/S0895435613004861).
