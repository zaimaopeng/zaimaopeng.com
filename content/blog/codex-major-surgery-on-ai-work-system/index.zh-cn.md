---
title: "我用 Codex 给我的 AI 工作系统做了一次大修补"
date: 2026-04-21T22:00:00+10:00
lastmod: 2026-04-21T22:03:00+10:00
description: "Codex 对我真正的价值，不是让我改文件更快，而是帮助我对一个正在运行的 AI 工作系统做高杠杆、有限边界的修补，包括 routing、worker 分工、validation loop 和执行可靠性。"
tags: ["AI", "Codex", "OpenClaw", "Agent", "架构", "工作流"]
categories: ["技术"]
showTableOfContents: true
slug: "codex-major-surgery-on-ai-work-system"
---

很多人现在对 Codex 的想象，还是“一个更快的 coding assistant”。

但这不是我用它最有价值的方式。

对我来说，Codex 最有价值的作用，是帮我给一个正在运行的 AI 工作系统做了一次受控的大修补。

不是 flashy demo，不是换个牌子的 autocomplete，也不是用它替代架构判断。

我真正需要的，是一个更窄但更有用的东西：它能帮我检查现状、修补有限边界、比较实现路径，并在我做结构判断时持续给 validation loop 施压。

## 系统不是在一个戏剧性的地方坏掉

真实运行中的 AI 系统，麻烦的地方往往不是某一个地方彻底炸了。

更常见的情况是：很多地方都半工作着。

工具是有的，一些 workflow 也已经能跑，助手也确实能做出有价值的工作。但这些能力外面的结构并不均匀。routing 是糊的，责任边界会漏，validation 不一致，有些路径是真的，有些路径则只是“看起来像真的”，直到你真的把它端到端跑一遍。

我当时面对的，就是这种状态。

所以我越来越觉得，做 AI 系统的 architecture work，很多时候更像外科手术，而不是从零发明。你不是总在空地上搭新东西，而是进入一个已经活着的系统，尽量保住那些已经能工作的部分，同时修掉那些持续制造混乱、漂移和假完成感的结构问题。

## 为什么 Codex 在这里有用

Codex 有用，不是因为它能替代 judgment。

而是因为这种工作特别受益于一个能快速看文件、打局部补丁、跨 surface 追踪流程、并持续推动 diagnosis 和 validation 来回闭环的搭档。

这里很多杠杆都来自“有限边界”。

我不是让 Codex 替我发明整个架构。我是在用它跑一类受约束的修补循环。

看这个 skill。补这个脚本。检查这个假设。对比这几份文件。再跑一次 validation。修掉这个不一致。然后再看下一层。

这个节奏很重要。

真正的价值不是“AI 生成了代码”，而是修补循环变得更紧、更容易持续下去。

## 到底修了哪类东西

这次反复出现的修补，大概有几类。

### 1. Routing discipline
很多看起来像执行弱的问题，本质上其实是 first-hop routing 错了。系统一开始就进错 lane，打开错 surface，或者一直停在 generic help mode，而不是进入正确 workflow。

### 2. Worker 和 parent 的责任分离
另一个反复出现的问题，是责任混线。如果 worker 在做重实现，parent 就应该负责 orchestration 和 integration，而不是半路又做一遍同样的事。如果 parent 负责 judgment 和 closeout，worker 就不该假装自己拥有整条 lifecycle。

### 3. Skill path repair
一个 skill 不是“磁盘上有个文件”就算真的存在。只有当 routing、references、helper scripts、validation steps 和状态语言在压力下都对齐，它才算真的能跑。

### 4. Validation 和 closeout 收紧
系统还需要更强的约束，来抵抗 fake progress。真实工作系统需要更明确的 done 标准、blocker 标准，以及哪些东西在被相信之前必须先验证。

## Codex 没有在做什么

这里也很重要，不要把它浪漫化。

Codex 没有独立设计整个系统。它没有替代 taste、judgment 或 responsibility。它也没有自己神奇地发现正确架构。

真正的杠杆来自：人的架构判断，加上一个能让受限修补循环快速持续下去的工具。

这个组合之所以重要，是因为 architecture work 很容易死在摩擦里。你看见一个弱点，但 tracing、patch、validation 的代价太高，于是系统就继续保持半乱不乱的状态。

Codex 降低了这个摩擦。

它让人能在设计压力还在的时候，继续往下修。

## 修完之后变了什么

结果当然不是完美。

但系统变得更一致了，更诚实地表达自己的状态了，也更容易把工作推进到真实里程碑，而不是停在一种表演性的“看起来在推进”。

一个系统不会因为它总能回答你，就自动变得值得信任。它更值得信任，是因为它的 workflow 更真实、ownership boundary 更清晰、validation loop 真的能闭合。

## 更大的教训

我越来越觉得，AI coding system 最被低估的用途之一，不是 raw code generation。

而是对一个正在运行的工作系统做迭代修补。

这和让模型帮你写一个孤立程序，是完全不同的 use case。它依赖 context、boundary、local feedback 和反复 validation。它更不像是在展示模型有多厉害，而更像是在构建一个 human-plus-agent loop，让它可以持续做外科式改动，同时不丢掉整个系统的形状。

这才是 Codex 对我真正的价值。

不是因为它写得更快。

而是因为它让我能做大修补，同时不假装病人很简单。
