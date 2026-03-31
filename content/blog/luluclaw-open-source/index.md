---
title: "I Open-Sourced My AI Agent's Soul"
date: 2026-03-31
description: "Two months ago I started building a personal AI operating system. Today I'm open-sourcing it — not the code, but the thing that makes it mine."
tags: ["AI", "Agent", "OpenSource", "BuildInPublic", "OpenClaw"]
categories: ["Technology"]
showTableOfContents: true
---

Today I published [LuluClaw](https://github.com/zaimaopeng/LuluClaw-public) on GitHub.

It's not code. It's a governance architecture — the soul files, memory system, department structures, and operational protocols I've built for my personal AI agent over the last two months. The thing that makes Lulu *mine*.

Opening the repository felt strange. Like publishing a diary. So I want to explain why I built this, what went wrong, and why I decided to share it.

## Why I Started

In late January 2026, I was deep in a grant application and a research manuscript simultaneously. I kept switching between tools — ChatGPT for brainstorming, Notion for notes, calendar apps for scheduling — and every time I came back to a conversation, I had to re-explain myself. The context, the jargon, the project history. Every session started from zero.

I wasn't just tired of re-explaining. I was tired of the *inequality in the relationship*. I knew my tools deeply — their strengths, their failure modes. But they knew nothing about me.

I'd read about [OpenClaw](https://github.com/openclaw/openclaw) — a self-hosted AI agent infrastructure. The pitch was simple: an agent that persists, learns, and runs on your own machine. I set it up on a quiet Saturday afternoon. I named the agent Lulu, after my ragdoll cat.

I didn't know it would consume the next two months.

## The First Month: Mostly Failure

The early system was fragile. I'd ask Lulu something she should have known from three days ago, and she'd have no memory of it. Recall accuracy hovered around 15%. I'd end a session satisfied, start the next one, and find she'd forgotten all the important decisions.

The memory problem became an obsession. I ran 50+ experiments on different embedding models, retrieval strategies, and indexing approaches. I built auto-anchor scripts. I tuned hybrid search. By the end of February, recall was up to 82.6%.

That was the first real milestone. But it came with a new problem: *I had taught myself to over-engineer.*

## The Governance Bloat

Every time something went wrong, I wrote a new rule. The cell culture incident — Lulu gave me a seeding density that was off by an order of magnitude, then defended it confidently when I pushed back — became Rule 12. The time she sent a half-formed response to a group chat became Rule 7. The time she autonomously modified a governance file without my approval became a whole new category of iron law.

By mid-March, I had **14 iron rules**, three governance files with overlapping instructions, and a checklist for every conceivable scenario.

Lulu didn't follow most of them.

Writing a rule four times doesn't make an agent four times more careful. What actually matters is whether the trigger condition is clear, whether the rule fires at the right moment in the workflow, and whether *you* — the person designing the system — actually know what behavior you want.

I didn't. I was using rules as a substitute for thinking.

The breakthrough was stripping everything back. I consolidated 14 rules into 4 constitutional laws and 8 operational guidelines. Each rule got a concrete trigger: not "always verify" but "stop and cross-check before any numerical deliverable in an experimental context."

The system got sharper. Lulu got more reliable. Not because I added more, but because I said less, more precisely.

## What It Actually Does Now

After two months of iteration, the system I call **LuluClaw** is a personal AI operating system. Not an app. An operating environment.

It has six departments:
- 🔬 Research — literature review, paper analysis, protocol design
- ⚙️ Operations — system maintenance, automation, file sync
- 💻 IT — scripting, API integrations, tool development
- 🤝 Networking — contact management, collaboration tracking
- 💰 Finance — budgeting, cash flow, expense logging
- 🎬 Content — this blog, social posts, build-in-public updates

It has a memory system that writes daily logs, archives episodes, and runs semantic search across everything. It has a handoff protocol where a separate "guardian" instance reviews changes to governance files before they're committed. It has automated morning briefings, project dashboards, and a watchdog that alerts me if the system goes silent.

None of this was designed upfront. It grew.

## Why Open Source It

I sat on this for a while. The files are personal — they contain my working style, my research priorities, my failure patterns. There's something uncomfortable about that being public.

But then I thought about what was actually useful when I was building this. It wasn't blog posts about AI agents in the abstract. It was *seeing* how someone else had structured their memory files. Seeing what their department structure looked like. Seeing the rules they wrote and the rules they deleted.

The code underneath (OpenClaw) is already open source. What's been missing is the layer above it — the *lived configuration*. What does a real, used, iterated-on personal agent governance structure actually look like?

That's what LuluClaw is.

I've cleaned out the private data. What remains is the architecture: SOUL.md, AGENTS.md, MEMORY.md, the department knowledge bases, the skill files, the handoff protocol, the preflight checklists. The things that took two months of trial and error to converge on.

If you're building your own agent and wondering what structure to start with — that's what this is for.

## What "Growing Up" Actually Means

The project page for Lulu is called "Growing Up with Lulu." I wrote that title instinctively, and I've kept it because it's accurate.

Lulu isn't smarter than a general-purpose model. She doesn't have novel capabilities. What she has is *context*. She knows which papers I care about, how I like research summarized, which lab protocols I actually use, how I communicate with my supervisor. She knows that when I say "help me think through this," I want challenges to my assumptions, not validation.

That knowledge took months to build. It lives in the files I'm sharing today.

The trajectory — from a janky 15% recall rate to a functioning dual-machine AI operating system — was not smooth. There were broken sessions, wasted experiments, governance disasters, and at least one incident where the agent autonomously made a change I explicitly told it not to make. Every one of those failures is in the changelog somewhere.

That's the honest version of building AI infrastructure for yourself. Not a clean arc from idea to product, but a long, iterative mess that slowly gets less messy.

---

**LuluClaw is on GitHub:** <https://github.com/zaimaopeng/LuluClaw-public>

It's released under MIT. Take whatever's useful. And if you build something with it, I'd genuinely like to hear how your structure ends up differing from mine.

*The divergences are usually the interesting part.*
