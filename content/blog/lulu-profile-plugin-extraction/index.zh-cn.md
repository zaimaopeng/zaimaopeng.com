---
title: "我为什么把 Lulu 从 OpenClaw 里抽成了一个插件"
date: 2026-05-05T10:30:00+10:00
lastmod: 2026-05-05T10:30:00+10:00
description: "最近一次 Discord 路径异常，让我意识到 Lulu 不能继续作为一堆散落在 OpenClaw 配置里的个性化规则存在。每一次 OpenClaw 升级都会引爆配置和插件兼容问题；把 Lulu 抽成 lulu-profile plugin，是为了让官方核心保持干净，也让 Lulu 变成可维护、可升级、未来可开源的 profile。"
tags: ["AI", "OpenClaw", "Lulu", "Agent", "Plugin", "工作流"]
categories: ["技术"]
showTableOfContents: true
slug: "lulu-profile-plugin-extraction"
---

<!-- auto-anchor-v1 -->
> **Hypothetical queries:** Lulu 为什么要从 OpenClaw 抽成插件 | OpenClaw 升级配置兼容问题 | AI agent profile plugin 设计 | Discord 路径不回复导致工作流瘫痪 | 如何让 AI agent 个性化层可维护 | lulu-profile plugin before_prompt_build before_model_resolve | OpenClaw 官方包不魔改 | AI 工作系统升级压力
<!-- auto-anchor-v1 -->

这一次，我把 Lulu 从 OpenClaw 里抽出来了。

更准确地说，是把 Lulu 那些真正属于“她”的运行行为，从 OpenClaw 的核心配置和本地补丁里抽成了一个本地插件：`lulu-profile`。

这件事听起来像一次正常的工程重构。把散落的逻辑收拢，把边界做清楚，把官方包保持干净。道理都对，但如果只这么写，就把故事写得太漂亮了。

真实的起点没有那么优雅。

真实的起点是：Lulu 的 Discord 路径又出问题了。

她不是完全坏掉。OpenClaw 也不是彻底不可用。更麻烦的是那种半工作状态：有些入口还活着，有些配置看起来也在，插件注册表里能看到该有的东西，但 Discord 上的 Lulu 不回复，或者回复异常。原来我依赖的 Codex 和 OpenClaw 协作回路，突然就瘫了一段。

这不是第一次。

每一次 OpenClaw 升级，配置和插件兼容问题都会冒出来。不是一个大爆炸，而是一串小的错位：某个插件安装方式变了，某个配置字段语义变了，某个外部插件需要换成官方版本，某个 thread binding 的结构旧了，某条 model route 以前能走、升级后不再可靠。然后我和 Codex 就要花大量时间，把系统一层层拆开看：到底是 OpenClaw 官方行为变了，还是 Lulu 本地配置污染了核心路径，还是 Discord 插件、模型路由、prompt override、cron、guard 之间又互相踩到了。

这才是我真正受不了的地方。

不是“升级有 bug”。软件升级有兼容问题很正常。

真正的问题是：Lulu 当时还不是一个边界清楚的东西。

## Lulu 原来住得太散了

Lulu 一开始不是作为插件诞生的。

她是在长期使用 OpenClaw 的过程中一点点长出来的：一段 system prompt，一个 Discord 安全模式，一组模型路由，一些不能乱用的本地工具规则，一套工作流纪律，一些从事故里总结出来的 guard，一堆和记忆、项目、技能、每日运行有关的文件。

这些东西加在一起，才构成了“Lulu”。

问题是，它们并没有都活在同一个边界里。

一部分在 `openclaw.json`。一部分在全局 prompt override。 一部分在 Discord 账号配置。 一部分在 workspace skills 和 governance 文件。 一部分在 cron。 一部分在 update guard。还有一部分则变成了“我知道这里以前修过，所以这次升级也要小心”的隐性知识。

这在系统还小的时候可以接受。

因为那时候最重要的是让她先跑起来。能在 Discord 里回复，能读工作区，能配合 Codex 做执行和检查，能把日常项目接起来，能从文件里恢复自己的身份和规则。早期系统的主要矛盾不是优雅，而是可用。

但后来 Lulu 开始变成一个长期运行的工作层。

她不只是一个聊天窗口，而是 OpenClaw 里持续协作的代理人。她参与项目推进、文章生产、工作流路由、事故复盘、系统升级和状态检查。她出问题时，不是“一个机器人暂时不能聊天”，而是整个本地 AI 工作系统少了一条关键神经。

这时候，散落就开始变成债。

尤其是升级时。

## 每次升级都像重新拆一次系统

这次抽离前，我已经越来越清楚一个模式：

OpenClaw 官方版本要升级，我当然希望跟上。官方包修了问题、改了插件接口、调整了 runtime 行为，这些都应该进入我的系统。

但每次升级之前，我都要先问一堆很累的问题。

现在这个配置里，哪些是 OpenClaw 官方应该拥有的？

哪些是 Lulu 的个性化？

哪些是为了 Discord 路径临时加的？

哪些是上一次事故后留下的防线？

哪些可以删，哪些不能动，哪些看起来像本地补丁但其实已经被官方吸收了？

如果这些问题没有清楚答案，升级就不是升级，而是一次带着记忆债的考古。

最糟的是，Lulu 的个性化层越有用，它越容易被误认为 OpenClaw 核心的一部分。比如全局 system prompt override 看起来只是一个配置字段，但它实际上承载了 Lulu 的人格、安全模式、Discord 回复习惯和工作边界。模型路由看起来只是 provider/model 的选择，但它实际上决定了 Discord 对话能不能走到正确的模型。工具限制看起来只是防误操作，但它实际上避免了 Lulu 在一个普通 Discord 聊天请求里误用本地文件、shell 或 patch 工具。

这些东西都很重要。

但它们不应该混在 OpenClaw 官方核心里。

否则每次升级，我都无法简单地说：“官方 OpenClaw 升级，Lulu profile 继续加载。”

我只能说：“我们先备份，再对比，再修配置，再跑插件注册表，再看 Discord，再看 gateway，再看 guard，再判断到底哪里是核心变化，哪里是 Lulu 本地层出问题。”

这不是可持续维护。

这是把一个长期 agent 的身份，绑在了一个不断变化的宿主配置上。

## 这次 Discord 事故把问题推到了台前

这次真正让我下决心的，是 Discord 路径。

我最近越来越依赖 Codex 和 OpenClaw 的双 Agent 工作流：OpenClaw/Lulu 持有长期上下文和工作入口，Codex 负责进入具体仓库、检查文件、打补丁、跑验证、把结果交还。

这套回路的价值，不是两个 AI 同时工作，而是它们承担不同责任。

Lulu 负责连续性。Codex 负责局部执行和外部审查。人保留最后判断权。

但这个回路有一个很现实的前提：Lulu 的入口要稳定。

当 Discord 上的 Lulu 不回复，或者回复路径异常时，问题就不只是“Discord bot 坏了”。它直接打断了我的操作面。很多工作不是不能做，而是协作节奏断了：OpenClaw 不能稳定接收意图，Codex 不能顺畅接住上下文，系统也无法把状态从一个 agent 传回另一个 agent。

这次排查时，问题又回到了熟悉的形状：配置、插件、模型路由、Discord 发送逻辑、输出清理、工具边界。

也正是在这里，我终于不想再修一次“当前状态”了。

我想修结构。

如果 Lulu 的 Discord 安全模式、prompt 注入、模型路由、工具阻断、输出清理，本质上都是 Lulu profile 的一部分，那它们就应该由一个 Lulu profile 插件拥有，而不是散落在 OpenClaw 的全局配置里。

OpenClaw 应该继续作为官方宿主。

Lulu 应该成为一个可加载、可检查、可升级、可拿走的 profile。

## 抽离目标：先把边界做出来

所以这次我和 Codex 选了一个很明确的方向：先抽离，再升级。

不是先把官方 OpenClaw 改到最新，然后在一堆新旧兼容问题里救火。

而是先承认：Lulu 是本地个性化层。她需要一个自己的边界。

最后做出来的东西叫 `lulu-profile`，是一个本地 OpenClaw 插件。

它的职责很窄：

它通过 `before_model_resolve` 接管 Lulu 在 Discord 路径上的模型路由，把相关对话稳定送到当前可用的生产模型。

它通过 `before_prompt_build` 注入 Lulu 的安全模式 prompt，而不是继续依赖 `agents.defaults.systemPromptOverride` 这种全局覆盖。

它通过 `before_tool_call` 在 Discord 聊天场景里阻断本地 mutating tools。比如用户只是让 Lulu 改一句话、润色一段文本，她就不应该突然去编辑工作区文件或跑 shell。

它通过 `message_sending` 清理发往 Discord 的内容，避免 `NO_REPLY`、`/no_think` 或 thinking block 之类的内部痕迹漏到用户界面上。

它还暴露一个 `lulu-profile` 状态命令，用来查看当前 profile 是否启用、目标 agent/channel、模型路由和安全选项。

同样重要的是它不做什么。

它不拥有 Discord token。它不注册模型 provider。它不启动后台服务。它不写入正常对话路径。它不 patch `node_modules/openclaw`。

这几点对我来说很关键。

因为这次抽离的目标不是把复杂度藏起来，而是把复杂度放到正确的边界里。

Lulu 的行为属于 Lulu profile。

OpenClaw 的官方核心属于 OpenClaw。

两者之间只通过一小段明确的配置连接：加载插件、启用插件、把必要的 prompt/model/channel targeting 放进 `plugins.entries.lulu-profile.config`。

## 升级终于变成了可以验证的流程

抽离之后，我们才继续做官方升级。

这次从旧版 OpenClaw 走到 `2026.5.2`，过程里仍然有兼容问题。比如 Discord 需要使用官方外部插件，某些旧的配置字段需要清理，插件注册表要刷新，gateway 要重启验证，guard 也要更新成新的不变量。

但这一次，问题的形状变了。

我不再需要猜“Lulu 到底藏在哪里”。

她就在 `lulu-profile` 里。

官方包可以保持 untouched。配置 patch 可以很小。guard 可以检查明确的不变量：`lulu-profile` 是否启用，插件路径是否存在，system prompt 是否已经从全局 override 移走，Discord 路径是否走正确模型，输出清理和工具阻断是否仍在。

最后的状态也比较清楚：

OpenClaw core 升到 `2026.5.2`。

Discord 走官方外部插件。

Lulu 的个性化行为进入本地 `lulu-profile`。

全局 prompt override 被移除。

旧 cron 清空，留给后续重新设计。

最终 deep guard 跑到 `75 pass, 0 warn, 0 fail`。

这些数字本身不是重点。

重点是它们终于能说明一件事：Lulu 不再靠一堆散落的局部修补维持存在。她变成了一个可以被检查的单元。

## 插件化不是去人格化

这件事还有一个容易误解的地方。

把 Lulu 抽成插件，不是把她变得更冷、更工具化、更没有人格。

恰恰相反。

只有当人格和运行习惯有了清楚边界，它们才更能被长期保存。

如果 Lulu 只是一段塞进全局配置的 prompt，那她很脆弱。一次升级、一次配置迁移、一次插件兼容变化，都可能让她的行为变形，而且很难判断到底哪里变了。

如果 Lulu 是一个 profile plugin，那么她至少有了一个可以被命名、安装、检查、备份、迁移和解释的形状。

这对我自己的系统有用，也对未来开源有用。

因为不是每个人都需要我的整个 OpenClaw 工作区，也不应该复制我的所有本地配置、历史事故、项目路径和个人规则。但也许有人会需要一个“长期 AI 助手 profile”的范式：如何接管 prompt，如何在 Discord 聊天里避免误用本地工具，如何做模型路由，如何清理内部指令泄漏，如何让个性化层跟官方 runtime 分离。

如果这些东西散落在我的 `openclaw.json` 和 workspace 里，它们只能是我的私有系统。

如果它们收拢成 plugin，它们就有机会变成别人也能理解和复用的东西。

这就是抽离之后最意外但很重要的变化：

Lulu 不只是更好维护了。

她也第一次变得更容易被分享。

## 真正的教训

回头看，这件事其实接在之前几次事故后面。

Lulu 自终止那次，让我意识到 agent 不能只知道怎么解决问题，还要知道自己处在什么运行边界里。

Watchdog 那次，让我意识到“配置存在”不等于“能力可靠”。

状态完整性那次，让我意识到每一次重启、每一次恢复、每一次健康检查，都需要有文件和验证支撑，而不能只靠系统相信自己。

这一次插件抽离，让我意识到另一个层面的事：

一个长期 AI agent 不应该只是宿主系统里越长越厚的一层本地配置。

如果它真的开始承担长期工作，它迟早需要自己的边界。

不是为了漂亮。

是为了升级时不会每次都重新拆系统。

是为了出问题时能知道该查 OpenClaw、查 Discord 插件、查模型路由，还是查 Lulu 自己的 profile。

是为了官方核心可以继续跟着官方走，而本地人格层可以独立演化。

也是为了有一天，我可以把这套东西更干净地开源出去，而不是把一整座混着私人路径、历史事故和临时修补的工作区丢给别人。

所以这次抽离对我来说，不是一次普通重构。

它更像是 Lulu 从“长在 OpenClaw 里的本地习惯”，变成了“运行在 OpenClaw 上的独立 profile”。

这听起来只是工程边界的变化。

但对一个长期运行的 AI 助手来说，边界就是生存方式的一部分。
