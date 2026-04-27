---
title: "为什么我选择 cleanup boundary，而不是彻底重写"
date: 2026-04-27T21:23:00+10:00
lastmod: 2026-04-27T21:23:00+10:00
description: "一次 planning-system cleanup 让我意识到，好的清理工作不一定是删掉所有旧名字，而是区分 active defects、legacy aliases 和 harmless history。"
tags: ["AI", "系统设计", "工作流", "Cleanup", "OpenClaw"]
categories: ["技术"]
showTableOfContents: true
slug: "cleanup-boundaries-over-rewrites"
---

有一种 cleanup 工作，从远处看起来特别明显。

系统里还残留一个旧名字。新名字已经成为 canonical。那任务似乎就是：把旧名字从所有地方删掉。

这就是 planner cleanup 一开始最诱人的版本。

但真正值得做的版本更有意思。

目标不是抹掉每一个 `superpowers` 字符串，而是稳定 `planner` 作为唯一 canonical callable planning layer，同时保留那些仍然有合理存在理由的旧引用：legacy alias、migration history、upstream lineage，或者 project-local evidence。

这个区别让整个工作变好了。

## 彻底删除不是正确目标

系统已经经历了一次 ontology shift。

`planner` 现在是 live planning layer。`superpowers` 仍然出现在旧表面、compatibility path、历史笔记和 upstream references 里。

天真的 cleanup 会把每一个残留字符串都当成 defect。

但这会制造错误激励。它奖励的是表面消失，而不是准确分类。它还可能破坏有用历史，或者隐藏系统为什么改名。

更好的问题不是“旧词还在哪里出现？”

而是“哪些出现仍然会影响当前行为或当前理解？”

## 分类让事情变得可控

真正有用的动作，是把残留引用分桶。

有些是 active defects：它们会让当前用户或 agent 学到错误的 canonical layer。

有些是 active-supporting surfaces：它们不是 runtime entrypoint，但仍然足够活跃，可能影响未来写作、检索或理解。

还有一些是 intentional keep-surfaces：compatibility shim、governance evidence、project history、upstream lineage name。

这个分类把一个模糊的 cleanup，变成了有边界的项目。

它也改变了这件事的感觉。任务不再是和每一个残留痕迹作战，而是做判断：保留能解释系统的东西，修掉会误导系统的东西，把无害历史留在原处。

## 边界保护了系统

一个关键决定，是暂时保留 `capabilities/methods/superpowers/`。

目录重命名的 blast radius 比当前问题本身更大。我们可以修正 live planning semantics，而不必强行做一次高风险物理迁移。

这是一个很有用的模式。

Cleanup 工作常常因为“改名更彻底”而显得更完整。但系统不会因为搜索结果里旧字符串更少，就自动更健康。

系统真正变健康，是因为当前行为更清楚，未来维护更不容易被误导。

有时这意味着重命名。有时这意味着把 legacy alias 标清楚，然后继续前进。

## 结果不是纯净，而是诚实

最后的验证结论不是“旧名字已经归零”。

它说的是：没有发现 remaining canonical-active runtime defect。那些仍然会教错旧 canonical name 的 active-supporting surfaces 已经清掉。剩下的命中集中在 compatibility shim、governance evidence、preserved lineage、project-local artifacts 和 historical references 里。

这是另一种 clean。

它不是纯净。

它是一张诚实地图。

对于一个长期运行的 AI workspace，一张诚实地图比一个表面干净的搜索结果更有用。

## 更大的教训

这个项目改变了我对 cleanup 的看法。

成熟的动作不总是删更多，而是选择和真实风险相匹配的边界。

如果旧引用影响执行，就修掉。如果它会让未来 agent 学到错误模型，就更新。如果它保留迁移语境或兼容性，就标清楚并保留。

好的 cleanup 不是追求一块完全空白的新地面。

它是判断哪些残留危险，哪些残留有解释价值，哪些残留根本不值得被升级成一场更大的迁移。

这没有彻底重写听起来戏剧化。

但它更可能让系统真的变好。
