---
title: "把个性化抽掉，OpenClaw 的治理才开始可移植"
date: 2026-05-07T19:30:00+10:00
lastmod: 2026-05-07T19:30:00+10:00
description: "我的 OpenClaw 工作区跑了大半年之后，越来越像一台为我量身打造的机器。但当我尝试把里面的治理体系给别人用时，第一反应——直接复制——立刻就错了。这篇是关于：为什么治理体系没法搬，能搬的是机制；以及我把自己 OpenClaw 拆成的七个治理插件。"
tags: ["OpenClaw", "AI", "Agent", "工作区", "插件", "治理", "基础设施"]
categories: ["技术"]
showTableOfContents: true
slug: "portable-openclaw-by-extracting-personalization"
---

# 把个性化抽掉，OpenClaw 的治理才开始可移植

我的 OpenClaw 工作区已经跑了大半年。

里面有 5 个 domain、几十个 skill、一份章程、几张路由表，以及一堆我自己反复试错出来的 playbook、模板、稽核脚本。它越来越像一台为我量身定制的机器：知道我用中文还是英文，知道我哪些文件不能给分身读，知道我什么时候要 deep-work、什么时候只想快速过一下信息流。

最近我开始好奇一个问题：

> 这套东西，能给别的 OpenClaw 用户用吗？

直觉的回答是：能啊，复制一份就行了。

但我刚开始动手就发现这个直觉是错的。

我能给别人复制的不是这一整个 workspace。能给别人复制的，是这套 workspace 里**抽掉个性化之后剩下来的部分**。这两个东西的差别，比我一开始以为的要大得多。

## "复制 workspace" 这个想法为什么不工作

最直接的方式当然是把整个 `~/.openclaw/workspace/` 打个包扔过去。

但这一打开就是一团乱：

- `domains/career/` 写满了我自己的研究方向、合作者、protocol。给别人看根本没意义。
- `CHARTER.md` 第一句就是 "彭在茂工作室"。还把 Lulu 当作我的云端心智来定义。
- `MEMORY.md` 里的"路由表"对应的是我特定的 5 个 domain，名字一改全部失效。
- `USER_PRIVATE.md` 是私密的，绝对不能流出。
- 所有 skill 的触发词都隐含着我的工作语言习惯。
- `runtime/` 全是我自己写的小 app 和 PowerShell 脚本，跟别人无关。

如果我硬把这堆东西丢给另一个人，他打开 OpenClaw 后第一时间会面对**一份别人的人生**。这不是治理体系，这是噪音。

但有趣的地方是：这堆东西**确实**包含了真正有价值的、可复用的东西。

只是它们和我的个人内容缠在一起。

## 真正能搬走的不是文件，是机制

把"workspace"和"治理体系"分开看，这一团乱就开始解开了。

我的 workspace 是**内容层**。具体的项目、具体的研究、具体的客户、具体的偏好。

我的治理体系是**机制层**。比如：

- "任何任务先识别 object type 再读最小相关表面" 这条三步路由
- "exactly 1 primary, 0-1 planner, 0-1 mode, ≤2 support" 这条 skill 加载守则
- "domains/, projects/, capabilities/, runtime/, archive/, memory/, inbox/" 这套 7 层物理布局
- "🔴 / 📋" 这两个旗帜
- "domain 有 6 种治理状态" 这套生命周期
- "每个项目都得有 PROJECT_LOG.md，收尾前更新" 这条 iron rule
- "每个治理文件顶部加 `<!-- auto-anchor-v1 -->` 块写假想查询" 这套召回增强约定

机制层根本不依赖我的具体内容。它是和我的人生**正交的**。

那么真正可以搬给别人用的，就是机制 —— 而我之前混着的，是把机制和我个人的实例化绑在了一起。

抽包的工程，本质上就是把这两层拆开。

## 抽包的 7 个产物

我用了一个下午把这件事做了：把治理机制从我个人 OpenClaw 里逐项抽出来，做成 **7 个独立的 OpenClaw 插件**。

| 插件 | 抽走的机制 |
|---|---|
| `openclaw-skill-taxonomy` | 7-kind skill 分类、selection guardrails、`skill-router` planner、`skill-author` 执行器 |
| `openclaw-workspace-ontology` | 7 层对象模型（domains/projects/capabilities/runtime/archive + memory/inbox）、object-type-first 路由、promotion rule |
| `openclaw-domain-ops` | domain 是什么、6 种治理状态、开/分/合/弃用/归档操作、域 README 模板 |
| `openclaw-project-ops` | 项目生命周期 + `/endproject` 命令 + 项目模板 + closeout / health-audit playbook |
| `openclaw-memory-anchors` | `<!-- auto-anchor-v1 -->` 协议、Hypothetical queries 写作指南、auto-anchor lint 脚本、episodic-memory 健康审计 |
| `openclaw-charter-pattern` | 6 段章程结构、🔴/📋 旗帜规范、daily/weekly briefing 骨架 |
| `openclaw-inbox-ops` | inbox CONTRACT（queue / mail / outbox 三 lane）+ intake / promotion playbook |

每个插件自带 `openclaw.plugin.json` 装包契约、独立 README、可拷贝的模板和脚本。装包者可以只装 `#1 + #2`（最小可用栈），也可以全装。

但**这些插件里没有任何属于我的内容**。

`CHARTER.md.tmpl` 里的"工作室名"是 `<<WORKSPACE_NAME>>`。`MEMORY.md.tmpl` 里的 domain 名是 `<<domain-1>>`。"权限结构"那张表里我的名字是 `<<PRINCIPAL_NAME>>`。`USER.md`、`IDENTITY.md`、`SOUL.md` 这种东西**根本就没有进抽包范围**。

抽包的过程一直在做一件事：每识别出一个机制，就立刻去找它在我个人 workspace 里的所有"实例化痕迹"，把它们一一替换成占位符。

如果做不到，那就说明这个东西其实不是机制，是个性化。它就不该进抽包。

## 个性化是摩擦力

我一开始以为这件事 80% 是工程整理。后来发现 80% 是**鉴别**。

对每一份治理文件，都要问一遍：

- 这一段是机制，还是个性化？
- 这条规则是普适的，还是只对我的工作模式有用？
- 这个 skill 名字是抽象角色，还是我自己叫熟了的快捷方式？
- 这条 iron rule 出自普适经验，还是我自己撞过的某个具体坑？

凡是答案偏向"个性化"的，要么剔除，要么明确标成 `<<placeholder>>` 让装包者自己填，要么挪到一个"参考用例"段落里说"这是 Michael 的具体做法，你可以参考但不要照抄"。

这件事做久了之后我体感到一个非常具体的事实：

> 个性化是摩擦力。

它不是一个"软性附加项"。一个治理体系里只要还残留着对原作者的人生的引用、对原作者工作流的暗示、对原作者具体工具栈的依赖，它的复用就会卡住。卡住的程度和残留量成正比。

很多 AI 圈里"我把我的 prompt / 工作流分享给大家"的尝试为什么效果有限？很大一部分原因就是没做这一层鉴别。分享出来的东西看起来 generic，但被另一个人装上之后会立刻 surface 一堆"这个名字是什么意思 / 这条规则为什么这么写 / 这个文件夹我没有"的钝痛。

抽包的工程，首要任务不是写新代码。是**剥离**。

## 一个反模式：直接 fork 自己的 workspace 给别人

抽包过程中我一直在心里跟一个反模式做对照。

那个反模式是：**把整个 workspace 打包成 starter template 给别人用**。

听起来很有诚意 —— 把我已经 battle-test 过的所有结构都送过去，让对方 fork 一份开始用。

但这个做法实际上是把"鉴别"这件事推给了装包者。装包者打开后会看到一大堆"看起来很专业但完全不知道哪些该改、哪些是机制、哪些是 Michael 的人生"，然后一项一项试出来。等他试明白，他基本也就重新走了一遍我抽包做的工作 —— 而且没有 manifest、没有 SKILL.md frontmatter、没有约束。

这个反模式之所以诱人，是因为它**对作者最省力**。

但它**对所有装包者都最费力**，每一个装包者都要重做一遍剥离。

所以正确的路是反过来的：作者把剥离这件事一次性做完，做成 manifest 化的插件分发，让装包者拿到的就是已经无个人痕迹的机制。

这个"作者多花一倍力气，让所有装包者都省力"的取舍，是任何想做"可移植 AI 治理"的人都得做的取舍。

## 这是 AI agent 时代的一种新基础设施

写到这里我想退一步说一个更大的事。

我做这件事的时候一直在体会到，OpenClaw 这种东西 —— 一个长期运行、跨多个工具、跨多个 agent 的本地 AI 操作系统 —— 正在变成一种新基础设施。它不像云服务那样集中，不像传统软件那样静态。它是**每个用户家里都长着一棵稍微不一样的树**。

每个人的 OpenClaw 都是个性化的。这没问题。

但如果"个性化"和"机制"长在一起，那这棵树就只对自己有用，没法跨用户复用。每个人都得重新种一棵。每个人撞过的坑都得自己再撞一遍。整个生态就 scale 不起来。

抽包是反着这个的力。

它假设：**机制是可以共享的，只是个性化掩盖了它**。

只要持续把个性化分离出去 —— 用 manifest、用占位符、用 plugin 边界 —— 那么不同 OpenClaw 用户之间就能在机制层共享，在内容层各自生长。这就有点像 Unix 系统：每个人 home directory 不一样，但 `/usr/`、`/etc/` 大家共享同一套机制。

这件事在 prompt 工程时代我们没认真做过，因为以前的 AI 是无状态的，每次都重启。

但现在 OpenClaw 这一层是**有状态的、有内存的、跑在你家里的、长期演化的**。它有 `domains/`、`projects/`、`memory/`、`inbox/`。它需要的不是更好的 prompt，而是**可复用的 governance 层**。

而 governance 想可复用，必须先把个性化抽走。

## 接下来

我自己的 OpenClaw 不会因为这件事变得不一样。该长什么样还长什么样。

但我现在多了一个发布表面。

下次另一个人想搭一个长跑的 OpenClaw，他不需要从零想"我怎么组织 domain"、"skill 之间会不会打架"、"项目收尾时该把什么提升到哪里"。他可以直接装这 7 个插件 —— 至少装最小栈的两个 —— 然后开始用，把自己的内容长在已经稳定的机制骨架上。

而我，把自己 workspace 里那些机制的部分抽走之后，反而第一次清晰地看见"我自己的部分"是哪些。

这一点比我预期的更有价值。

剥离的过程本身，就是对自己工作系统的一次诚实审计。

---

*抽包后的 7 个插件、INVENTORY 与 DESIGN 文档现在都在我本机的 `~/.openclaw/plugin-extraction-2026-05-07/` 下。如果你也在跑 OpenClaw 想试一下，可以从最小可用栈（`openclaw-skill-taxonomy` + `openclaw-workspace-ontology`）开始装。*
