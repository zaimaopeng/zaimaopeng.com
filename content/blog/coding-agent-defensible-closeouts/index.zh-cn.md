---
title: "怎样让 coding agent 的 closeout 真的站得住"
date: 2026-04-27T21:22:00+10:00
lastmod: 2026-04-27T21:22:00+10:00
description: "coding agent 的 closeout 标准只有在任务状态、review gate、验证证据和 residual risk 汇报彼此支撑时，才真正可信。"
tags: ["AI", "Coding Agent", "验证", "工作流", "OpenClaw"]
categories: ["技术"]
showTableOfContents: true
slug: "coding-agent-defensible-closeouts"
---

Coding agent 任务里最难的部分，很多时候不是写 patch。

而是判断这件事是不是真的完成了。

任务一小，这听起来很简单。任务一大，就会牵涉测试、review、delegated subtasks、部分失败、retry、dirty worktree，以及用户真正需要关注的风险。到了这个时候，“done” 会变成一个很软的词。

`coding-agent-closeout-hardening` 这个项目，就是为了让这个词不那么容易被误用。

## 自信不是 closeout

Coding agent 可以在工作真正完成之前，就听起来已经完成了。

它可以解释改动，可以提到测试，可以写出整齐的总结，也可以说一些关于 residual risk 的正确话。

但如果整个工作流没有强制证据对齐，这些都不够。

一个可信的 closeout 需要回答具体问题：

- 改了什么？
- 验证了什么？
- 什么失败过，又怎么修掉？
- delegated review 有没有通过？
- 还剩什么风险？
- 人接下来最应该看哪里？

没有这个形状，最终回答就很容易变成一种自信表演。

## 契约不能只靠一个 skill

这轮工作里一个重要发现是，`coding-agent` skill 不能单独承担全部 closeout 标准。

Closeout 行为还依赖相邻表面：deep-work 的 outcome semantics、delegated review 的预期、以及决定什么时候需要更重 review path 的 routing playbook。

所以这次 hardening 对齐了多个表面。

Coding-agent contract 更明确地要求任务状态、基于证据的验证语言、delegated review gate、residual risk，以及最终回答结构。相邻方法也被同步收紧，让“done”的定义不只存在于 executor 自己身上。

这很重要，因为工作流最容易在连接处失败。

如果一个表面说 review failure 是真实状态，另一个表面却把它当成小建议，agent 很容易滑回过度乐观的完成语言。

## 验证让它变成真的

这个项目没有停在 doctrine edits。

它做了 regression pass、带 negative control 的 live validation slice、bounded full rework-loop pressure test，以及 multi-file benchmark。其中一轮还真的发现并修掉了 cross-surface mismatch。

这就是它从“写了更好的规则”变成“规则被真实压力测试过”的时刻。

系统不只是在说“我们应该更好地结束任务”。它有了被压测、被观察、被调整过的例子。

这种验证会改变 final report 的含义。它让 agent 说“这个 phase complete”时，背后有证据链，而不是只有语气。

## Review failure 必须可见

我特别在意 `review_failed` 这个状态。

如果 delegated review 发现了真实问题，parent task 不应该把它悄悄抹平成“基本完成”。它应该内部识别这个失败并进入 rework，或者明确告诉用户失败在哪里、下一步是什么。

这是很小的语言差异，但后果很大。

当 review failure 消失时，人就失去了理解真实状态的机会。Agent 也许最后还是会修掉问题，但系统已经在训练自己：失败的 review 不是一等事件。

这正是 closeout 开始不可信的方式。

## 我想要的标准

我想要的不是夸张的谨慎。

我不希望每个 final answer 都变成法律备忘录。我想要的是和任务规模、风险相匹配的简洁真实 closeout。

小改动可以只需要简短总结和一行验证。

大一点的 coding task，就应该说明关键改动、做过的验证、剩余风险，以及如果还有事情没完，人接下来应该看哪里。

重点不是字多。

重点是站得住。

当 agent 说一个 coding task done 的时候，这句话背后应该有足够结构，让另一个 agent 或人不用重放整段会话，也能理解它为什么成立。

这就是这次工作改变我判断的地方。

最终回答不是因为听起来漂亮，就代表工作结束。

只有当证据、review 状态和剩余风险彼此一致时，工作才真的结束。
