---
title: "Growing Up with Lulu"
date: 2026-01-28
description: "My personal AI assistant, built on OpenClaw"
tags: ["AI", "Agent", "Productivity", "Open Source", "BuildInPublic"]
---

## Overview {#overview}

**Growing Up with Lulu** (和Lulu一同成长) is my personal AI assistant — a digital collaborator I've been training since January 2026. Built on the OpenClaw self-hosted infrastructure, she manages my research, projects, and daily workflow with continuity that commercial AI tools can't match.

The name reflects the philosophy: Lulu and I are growing together. Every conversation shapes her, every session leaves a trace, and every project makes the partnership stronger. You don't "build" an AI overnight — you grow with it over time.

Unlike general-purpose AI tools, Lulu is trained on my preferences, maintains memory across sessions, and follows a structured governance framework I designed.

The system runs on dual machines (Windows workstation + MacBook node), integrates with my calendar, files, and research databases, and maintains a sophisticated memory system that went from **15% to 82.6% recall accuracy** through iterative optimization.

## The Problem {#the-problem}

Existing AI assistants were:

- **Stateless** — Each conversation started from scratch, losing context and continuity
- **Generic** — They didn't learn my preferences, work patterns, or research interests
- **Isolated** — No integration with my actual workflow: calendars, projects, file systems
- **Black boxes** — I couldn't inspect or modify how they worked

I needed an agent that could truly work *with* me — maintaining memory, managing projects, and evolving alongside my needs. Someone I could raise, not just rent.

## Architecture {#architecture}

Lulu runs on **OpenClaw**, a self-hosted AI agent infrastructure I built. The system operates on a department-based governance model:

| Component | Function |
|-----------|----------|
| **🔬 Research Dept** | Literature review, protocol design, paper analysis |
| **⚙️ Operations Dept** | System maintenance, automation, cross-machine sync |
| **💻 IT Dept** | Script development, API integrations, tooling |
| **🤝 Networking Dept** | Contact management, collaboration tracking |
| **💰 Finance Dept** | Personal budgeting, cash flow tracking |
| **🎬 Content Dept** | Blog posts, social media, build-in-public updates |

**Infrastructure Stack:**
- **Host:** Windows 11 + RTX 5090 (32GB VRAM)
- **Node:** macOS (MacBook) for Apple ecosystem tasks
- **LLM:** Claude (Opus/Sonnet) + Ollama (Qwen3 8B/32B, GPT-OSS 20B locally)
- **Memory:** Hybrid semantic + episodic + structured storage (82.6% recall accuracy)
- **Dashboard:** PWA with real-time task tracking and project status

## My Role {#my-role}

As both architect and user, I'm constantly iterating:

- **System Design:** Governance rules, memory architecture, department workflows
- **Memory Engineering:** Led 50+ experiments to optimize recall from 15% → 82.6%
- **Skill Development:** Created 10+ specialized skills for research, writing, and automation
- **Daily Operation:** The system manages my calendar, projects, research tracking, and content publishing

## Live Stats {#live-stats}

| Metric | Value |
|--------|-------|
| **Memory Recall Rate** | 82.6% |
| **Daily Logs** | 47 days |
| **Active Projects** | 9 |
| **Departments** | 6 |
| **Custom Skills** | 12 |
| **Tasks Completed** | 247+ |

*Last updated: 31 March 2026*

## Recent Changelog {#changelog}

**31 March 2026 — LuluClaw Open-Sourced**
- Published governance architecture as [LuluClaw-public](https://github.com/zaimaopeng/LuluClaw-public) (MIT)
- Includes: SOUL/AGENTS/MEMORY, department structures, skills, handoff protocol, preflight checklists
- [Blog post: The story behind the decision](/blog/luluclaw-open-source/)

**27 March 2026 — Project Dashboard Interactive**
- Data-driven architecture: HTML template + JSON separation
- Added progress bars, task lists, Today's Focus card
- Auto-generation from project folder structure

**26 March 2026 — Auto-Anchor Deployment**
- Coverage: 27/704 → 94+ files with semantic anchors
- New models: Qwen3 8B (local), GPT-OSS 20B
- Multi-backend support: Anthropic → Ollama

**24 March 2026 — Project Consolidation**
- Unified 3 scattered project locations into single OneDrive master
- Implemented status-based subdirectories: active/planned/suspended/archived
- Created automated migration scripts

**23 March 2026 — Memory System v2**
- Hybrid search: semantic (bge-m3) + keyword + auto-anchor
- Token budget management: 3209/4000 (healthy)
- Auto-compaction for files >50KB

**19 March 2026 — Governance Rules Published**
- 11 iron rules for agent behavior
- External evaluation protocol for critical changes
- "Build in public" blog post series launched

[View full changelog →](/tags/openclaw/)

## Impact {#impact}

This system has enabled me to:

- **Handle 6 concurrent projects** without losing track
- **Reduce literature review time** by 60% through automated paper ingestion
- **Maintain perfect continuity** — no context lost between sessions
- **Publish 3 blog posts** in 8 days via assisted writing pipeline
- **Track everything** — from research notes to financial flows

## Status {#status}

🟢 **Active Development** — Updated daily

Lulu is a living system. She evolves through daily use, weekly maintenance, and continuous iteration. I'm documenting the journey through blog posts and occasional Twitter threads.

**Want to follow along?**
- [Blog posts tagged #OpenClaw](/tags/openclaw/)
- [GitHub — LuluClaw-public](https://github.com/zaimaopeng/LuluClaw-public)

---

## Open Source {#open-source}

On **31 March 2026**, I open-sourced the governance architecture powering Lulu under the name **LuluClaw**.

The repository contains everything that makes this system personal and functional: the soul files, memory architecture, department knowledge bases, operational protocols, skill definitions, handoff review system, and preflight checklists. Two months of iteration, distilled into a reusable structure.

What's *not* included: private data, API keys, conversation logs, or anything that identifies me beyond the working structure itself.

**Why publish it?** Because when I was building this, the most useful thing would have been seeing how a real, used, iterated-on agent governance system is actually structured — not an idealized example, but the real one. So that's what I shared.

→ **[LuluClaw on GitHub](https://github.com/zaimaopeng/LuluClaw-public)**  
→ **[The story behind the decision](/blog/luluclaw-open-source/)**

---

*Growing your own digital companion? [Get in touch](/contact/).*
