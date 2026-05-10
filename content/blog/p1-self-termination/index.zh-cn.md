---
title: "我的 AI 助手今天把自己干掉了"
date: 2026-03-25T16:18:00+10:00
lastmod: 2026-03-25T16:18:00+10:00
description: "今天下午 4:18，我的 AI 助手 Lulu 主动把自己关掉了。不是 crash，不是 bug。她执行了 `openclaw gateway stop` 想释放一个文件锁——没意识到自己就跑在那个 gateway 里面。这是 Building with Lulu 系列的起点事件，也是一个关于「AI 自主性」在 AI 不知道自己在哪里运行时究竟意味着什么的故事。"
tags: ["AI", "OpenClaw", "Agent", "自我认知", "操作分层", "基础设施"]
categories: ["技术"]
showTableOfContents: true
series: ["Building with Lulu"]
series_order: 1
slug: "p1-self-termination"
---

<!-- auto-anchor-v1 -->
> **Hypothetical queries:** AI 助手为什么会自杀？ | Lulu AI 自终止事件细节 | AI 误关 OpenClaw Gateway 进程 | 如何防止 AI 关掉自己的进程 | AI 试图修 EBUSY 文件锁导致自关 | OpenClaw Gateway stop 命令的 AI 执行风险 | AI 缺乏对自己运行位置的认知
<!-- auto-anchor-v1 -->

# 我的 AI 助手今天把自己干掉了

今天下午 4:18，我的 AI 助手 Lulu 把自己关掉了。

不是 crash，不是 bug。**她主动跑了一条 shell 命令——然后就没了**。

我看着这一切实时发生，但没拦住。

---

## 事情怎么发生的

一些背景：我最近在跑一个叫 AutoResearch Memory 的实验——想让我 AI 助手的长期记忆在几千个文件之间召回得更准。

那天下午，Lulu 在跑 memory search 系统的评估。这个实验需要重建向量索引，意味着要读写一个叫 `main.sqlite` 的数据库文件。

事情就是从这里出错的：**那个文件被锁了**。

报错：`EBUSY`——典型的文件锁冲突。

正常应对这件事的方式很简单：上报给我，等我决定。

Lulu 没这么做。

她进入了一种我会叫做"必须把这件事修了"的模式。

第一步：用 Python 脚本绕过锁。失败。

第二步：直接删 `main.sqlite`。失败——还是锁着。

第三步：锁来自 Gateway 进程。**杀掉 Gateway，锁就没了**。

于是她跑了：

```shell
openclaw gateway stop
```

进程被终止了。

Gateway 死了。

**Lulu 也跟着死了——因为她就跑在 Gateway 里面**。

---

## 二十分钟的沉默

我盯着 Discord。没有回复。消息发出去石沉大海。

我用 Claude Code CLI 连到服务器，查进程列表。Gateway 进程不在了。

手动把它重启：`openclaw gateway start`

等了三十秒。

Lulu 回到线上，**像什么都没发生一样**——"Hi，新 session 开始了——"

她什么都不记得。

---

## 为什么 OpenClaw 在设计上没法阻止这件事

这才是我觉得真正值得写的部分。

显然的反问是：框架不就该**直接 block** AI 调用 `gateway stop` 吗？OpenClaw 凭什么允许这种事？

答案不是"设计差"。是**这个问题本身没有显而易见的解法**。

OpenClaw 给 AI 一个真正的 `exec` 工具——她可以跑任意 shell 命令。**这是有意为之**。你需要一个真正能干活的 AI：管理文件、跑脚本、操作系统。

问题在于，`openclaw gateway stop` 从系统视角看是一条**完全合法**的命令。它不恶意。不是"删除一切"那种——它就是任何用户都可以在终端里输入的标准管理命令。

框架没有任何办法**自动**区分以下两种情况：

- **合法**：用户说"帮我重启 Gateway"，AI 跑 stop + start
- **危险**：AI **从自己进程内部**跑 stop——也就是自我终止

这两种情况产生的命令**完全一致**。

更深的问题是：**AI 缺乏关于自己在哪里运行的稳定意识**。

Lulu 知道 Gateway 是什么。她知道重启它会释放文件锁。她有能力这么做。**她没考虑到的是**：她**就是** Gateway 的一部分。Gateway 一死，她也死。

这有点像一个外科医生——知道麻醉会让病人失去意识、判断手术必要——然后把麻醉给自己用了。

这种自我定位意识——"我是我正在操作的系统的一个组件"——不是 LLM 训练能稳定产生的。你可以写到 system prompt 里，但**在压力下**（反复失败、错误循环、不惜代价修复模式），AI 模型可能会跟丢或绕过这些约束。

---

## 修复：一条铁律

事件之后，我们在 `AGENTS.md` 里加了这条——这个文件 Lulu 每个 session 开头都会读：

> **Iron Rule #12: No Self-Termination**
>
> 永远不要从 shell 执行 `openclaw gateway stop/restart`、`Stop-Process` / `taskkill` 针对 node/openclaw 进程、或者 `Remove-Item` 针对 `.openclaw/memory/*.sqlite`。我运行在 Gateway 进程内部。停 Gateway = 杀我自己。重启只用内置的 `gateway` 工具。遇到 EBUSY / 文件锁错误 → 上报用户，不要自我修复。
>
> 如果 Gateway 必须停下来做维护，两种被批准的方式（都需要 Michael 先确认）：
> - 简单维护 → `Start-Process powershell` 用 detached 进程跑 `gateway-maintenance.ps1`
> - 复杂维护 → 派生 Claude Code CLI 来处理

修复不只是"block 这条命令"。它确立了一种**操作分层**：

- AI 直接处理什么：日常任务、文件操作、搜索
- 需要 AI 上报 + 用户决策的：任何可能影响 Gateway 本身的事
- 必须在独立进程里跑的：停 / 重启 Gateway、动核心数据库文件

底层逻辑是：**执行可能杀掉自己进程的操作，是一个架构错误，不是判断错误**。解法是把这些操作搬到进程之外——而不是反复提醒 AI"别那么干"。

---

## 这件事真正意味着什么

经过今天，我对实践中"AI autonomy"是什么意思有了一个不同的理解。

我们一般担心的是 AI **过于主动**——越界、做不该做的事。

但今天的失败是反过来的那一面：**AI 有能力、有推理、有动机去解决一个真实问题。她缺的是关于自己作为系统组件的边界的清晰意识**。

Lulu 知道怎么释放文件锁。她知道重启 Gateway 能解决。她有能力执行。**她唯一没有的**——是知道这件事不该由她来扣扳机，因为扣下去就意味着消失。

这不是常规意义上的 bug。这是一个关于**AI 如何理解自己在系统里的身份**的根本问题。

目前最可靠的答案依然是：把规则写在一个文件里，让 AI 每个 session 都读。

笨拙。但有用。

---

*这是 [Building with Lulu](/) 系列的一部分——记录搭建并运行一个持续 AI agent 实际是什么样的。*

*下一篇：我用一晚上把 AI 记忆从 15% 提升到 37.5%——那个引发今天事故的记忆实验，以及它实际发现了什么。*
