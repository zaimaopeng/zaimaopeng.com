---
title: "那个 ECONNREFUSED 死循环，给我的工作站迁移做了一次审计"
date: 2026-05-10T01:30:00+10:00
lastmod: 2026-05-10T01:30:00+10:00
description: "几周前我跟 AI 说工作站已经'完全搬到 Windows 这边了'。是搬完了——只是搬完了我看得到的那一层。前一天晚上一个 ECONNREFUSED 死循环逼出了一次彻底审计，浮出 13 件事还连着旧设置：孤儿服务、未注册的兄弟 agent、断掉的 skill 路径、明文 key、版本落后的模型。迁移不是停手那一刻就算完，是 audit 全绿那一刻才算完。"
tags: ["OpenClaw", "AI", "迁移", "审计", "工作区", "基础设施"]
categories: ["技术"]
showTableOfContents: true
slug: "econnrefused-audited-my-migration"
---

# 那个 ECONNREFUSED 死循环，给我的工作站迁移做了一次审计

几周前我退掉了 MacBook 作为 OpenClaw 的工作机，把所有东西收束到一台 Windows workstation。我跟 AI 说："Mac 那边完全没了，那个 node 配置不用再留着。"我清掉了我以为相关的几块东西，然后就翻篇了。

直到前一天晚上我给它甩了一张截图。

截图是 OpenClaw gateway 的控制台，无限往下滚：

```
node host gateway connect failed: connect ECONNREFUSED 100.76.94.64:18789
node host gateway closed (1006)
node host gateway connect failed: connect ECONNREFUSED 100.76.94.64:18789
...
```

我让它修这个 bug。

我以为：找到出问题的进程、杀掉、收工。十分钟的事。

实际：六小时、十三处修复，以及一条很清晰的结论——**这不是一个 bug，这是一笔被推迟的 audit，到点结账了**。

## 我为什么以为已经搬完了

迁移那阵我的心智模型很简单："重要的东西现在都跑在 Windows 上了；Mac 已经下线了；workspace 文件全在这边。"

这个模型是对的。它也是不完整的。

迁移触及的是我**看得到的那一层**——文件、项目、活的 gateway、我每天用的 skill。它下面还藏着第二层：我用了一年攒下来的**假设**：

- Windows Startup 文件夹里一个开机自启的 `node.cmd` 快捷方式（Mac 时代的 headless worker），每次登录都被唤起
- 一个 `node.json` 配置，指向**旧的** Tailscale 端点
- 一个后台跑着的 skill mirror watcher 守护进程，维护着早已没人读的 junction
- 一些放在 `agents/` 下、但从来没在 `agents.list` 里注册过的 agent 目录
- 主配置里明文存着的 API key，还有 9 份带着 key 的 `.bak` 滚动备份
- 默认模型还停在比官方推荐落后一个小版本的位置

这些东西**单独看**都"还行"。系统照常跑。没一个会响到能让我注意到。

直到其中一个真的响了。

## 把线头拉出来的那个 bug

那个 ECONNREFUSED 死循环，原因就一个：那个孤儿 `node.cmd` 脚本。它每次登录被自动唤起（一个我从来没注意到的 Startup folder 启动项），读着一份指向**本机** Tailscale IP + 18789 端口的旧配置，去尝试连接。

gateway 确实**就在**这台机器上、确实**正在**监听 18789——但它绑的是 `127.0.0.1`，不是 Tailscale 接口。所以每次连接都被拒。脚本重试。无限。

按窄义来说，这个 bug 的修法只需要四条命令：杀进程、`openclaw node uninstall`、删 Startup 快捷方式、重启 gateway。十分钟搞定。

但我刚开始看，pre-flight 的 `openclaw doctor` 就已经在喊另外一堆毫不相关的问题：*Codex 兄弟 agent 在磁盘上但没注册到 `agents.list`。`plugins.allow` 是空的。`groupAllowFrom` 缺。一堆孤儿 transcript。明文的 API key。*

每一条往回追，都来自同一个时段——那次迁移的窗口期。没有一条被审计过。

于是我停下"修这个 bug"这件事，开始**拉这根线头**。

## 浮出水面的十三件事

| # | 问题 | 迁移期的成因 |
|---|---|---|
| 1 | `node.cmd` 通过 Startup 文件夹无限循环 | Mac worker 退役写了脚本，但 Windows Startup 那个快捷方式是更早一次测试时建的，从来没被清理 |
| 2 | Discord 流式动画是关的 | 沿用自之前流式 UI 不重要的设置——但 Discord **现在已经是**主交互面 |
| 3 | `plugins.allow` 白名单是空的 | bundled provider discovery 收紧后没人跑迁移把 allowlist 填上 |
| 4 | 28+ 个自定义 skill 被 symlink-escape 静默拒绝 | skill loader 加固后会拒收跳出 root 的 symlink；workspace 的 mirror 布局正好踩中 |
| 5 | 9 个 skill 在两个位置各有一份 fork | 两阶段 refactor 留下了旧 `~/.openclaw/skills/` 和新 `workspace/capabilities/skills/` 两份都被当 canonical 维护 |
| 6 | Codex 兄弟 agent 没注册 | `agents/codex-ops/` 和 `agents/codex-worker-harness/` 有 auth 状态、有 sessions，但 `agents.list` 里没条目 |
| 7 | 245 个孤儿 transcript 文件 | sessions 索引里删了，磁盘上没删 |
| 8 | `cron/jobs.json` 重置了，但历史 run log 还在 | 手动清理时漏了 `runs/` 目录 |
| 9 | 主配置里明文的 `VENICE_API_KEY` 和 `ANTHROPIC_API_KEY` | 初始化向导是明文写的；迁移到 env var 间接引用的工作被排过期但没执行 |
| 10 | OAuth token 明文，复制到 9 份滚动 `.bak` 里 | 自动备份把泄漏放大了 |
| 11 | `bundledDiscovery` 卡在 legacy compat | schema 升级时有过 "compat" 这个中间档；没人逼它升到 "allowlist" |
| 12 | 默认模型停在 `gpt-5.4`，但官方建议早就指向 `gpt-5.5` | 模型升级被跳过——因为没东西在响着喊 |
| 13 | 一个死掉的 skill-mirror-watcher Python 守护进程 | 在后台没人盯着跑了几个礼拜，维护着没人读的旧 mirror |

六条是安全 / 正确性问题。五条是性能 / 体感。两条纯卫生级。**没有一条**是我汇报的那个 bug 本身。

## 模式：在我看得见的边界处宣布"完成"

我一直回到同一个观察上：当时我把迁移宣布为"完成"，是在我**看得见的边界**上宣布的。文件搬完了。Mac 关机了。在用的界面照常工作。

但长期跑的 AI 系统有**两层边界**。

可见层——跑着的 gateway、开着的 chat、按需触发的 skill。这些都在工作，你就说"完成"。

不可见层——所有**在不同条件下才会重要**的配置。只在登录时唤起的开机自启脚本。只在 gateway 挂掉时才醒来的看门狗。只在恢复时才被读的备份文件。只在被显式调用时才浮出问题的 doctor 检查。只 warn 不 block 的 schema 校验。

不可见层比可见层**宽得多**。在一个跑了一年的 AI workspace 里，它每个月都更宽一点。

可见层在工作时，不可见层是**沉默**的。可见层一旦坏了——**那就是**不可见层亮相的时候。以错误循环的方式。以 doctor 警告的方式。以负载下的奇怪行为的方式。

那个 bug 报告，不过是不可见层不再不可见的那一瞬间。

## 迁移是一次 audit，不是一个时刻

我从这件事里带走的心智模型：

> 迁移不是 cutover 那一刻就算完成的。是当一次 audit 把可见和不可见的全部状态都走了一遍并且报告全绿时，才算完成。

具体到 OpenClaw 这种本地 AI 系统，**post-migration audit checklist** 大致是这样：

- `openclaw doctor` —— 干净
- `openclaw security audit` —— 干净
- `agents/` 目录的实际内容和 `agents.list` 一致
- `cron/`、`flows/`、`delivery-queue/` —— 没有孤儿
- 所有明文凭证都迁到 env var 间接引用
- 所有 `.bak` / `.clobbered` / staging 快照都被审过有没有泄漏的密钥
- 所有 skill 路径都真正在加载（没有静默的 symlink-escape 拒绝）
- 所有计划任务和 Startup folder 启动项都是已知且有意的
- 默认模型和插件 allowlist 跟当前官方建议对齐
- 所有 `nodes/` 和 `devices/` 的配对记录都和当前在用的 worker 匹配

任何一条是红的，迁移就**还在进行中**——哪怕可见的那一面看起来一切正常。

"东西大体在跑"和"东西真正干净"之间的那段工作，就是成本所在的地方。它不算大，但是真实存在的，我把它当作沉默的技术负债已经背了好几个礼拜。

## 反模式：信赖 manifest

我想专门点出一个我自己掉进去过的失败模式。

我跟 AI 说"Mac 完全没了"的时候，我也更新了 `CLAUDE.md` 反映这件事。我感觉这件事被记录下来了。AI 能读到。doctrine 是当前的。

但**那份 doctrine 是 manifest，不是 state**。它描述的是我**意图**中的世界。它没有审计我**实际有的**世界。

这件事危险的版本是：manifest 被更新了，然后写的人开始**拿 manifest 当 verifier 用**。"我写了 Mac 没了，所以 Mac 没了。"但 manifest ≠ system state。迁移是按 system state 判，不是按 doctrine 判。

修法不是停止写 manifest——它们是有用的。修法是：**迁移之后，audit 跑的对象是系统，不是 manifest。** 而能合上一次迁移的，是 audit，不是 manifest 更新。

## 接下来

我的 OpenClaw 现在的状态我会形容为"audited clean"，而不是只是"running"：

- Doctor：零警告（一条信息级提示）
- Skills：41 个 eligible，单一 canonical 家，没有 fork
- Agents：磁盘上的目录全都注册了，没有孤儿
- 密钥：任何配置或备份里都没有明文
- 模型：跟官方最新建议对齐，吃到了平台为这个模型专门做的 parity 修复
- 备份：scoped、带时间戳、随时能回滚

更重要的是我现在握住了**那张 audit checklist 本身**。下次再迁移点什么——一台工作站、一个 domain、一个子 agent——那张 checklist 会作为迁移的一部分跑，而不是等下一个 bug 浮出来再跑。

我想让自己内化的那条纪律：

> 没跑 audit，就不算迁移完。

这不是一条生产力小贴士。这是长期跑的系统怎么保持干净的方式。每一次没跑 audit 就说"我们差不多搞定了"，都只是一笔被推迟的 bug——而推迟会**收利息**。

---

*这次 audit 是通过 Windows 工作站上的交互式 Claude Code 驱动的，对接的是 `127.0.0.1:18789` 上的 OpenClaw gateway。总耗时约 6 小时引导式调查；动到的：13 处不同修复、约 10 MB 旧物清理、41 个活跃 skill 单一来源整合。*
