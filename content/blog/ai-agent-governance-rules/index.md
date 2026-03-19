---
title: "I Had 14 Rules for My AI Agent. Most Were Useless."
date: 2026-03-19
description: "How I went from governance bloat to a system that actually works — lessons from two months of running a persistent AI assistant."
tags: ["AI", "Workflow", "Productivity", "Agent"]
categories: ["Technology"]
showTableOfContents: true
---

Two months ago, I set up a persistent AI assistant called **Lulu** on [OpenClaw](https://github.com/openclaw/openclaw) — named after my ragdoll cat, with the persona of a cat wearing a lobster shell. She lives on Discord, and on a good day, she helps me draft research papers, discuss manuscript frameworks, manage shrimp-keeping data for my hobby blog, and handle all sorts of research admin.

She has three core governance files: a personality file, an operations manual, and a long-term memory file. Over time, I kept adding rules to these files. Every mistake got a new rule. Every near-miss got a safeguard. By March 2026, I had **14 iron rules**, a monolithic preflight checklist, and the same instruction written in three different files.

Lulu wasn't following most of them.

## The Incident That Started the Rule Bloat

In mid-March, Lulu gave me cell seeding densities that were off by an order of magnitude — 10⁴ instead of 10⁵. I noticed something felt wrong and pushed back. She confidently defended her answer. I pushed back again. She doubled down. It wasn't until I got visibly frustrated that she actually re-examined her assumptions — and found the error.

By then, an entire batch of MG-63 cells was wasted.

My response? Add a rule: "Cross-validate experimental parameters against known reference values." Reasonable. But then I added the same rule to the personality file, restated it in the memory file, and wrote a dedicated checklist section.

**The same rule, written four times, in four files. And the agent still didn't reliably follow it.**

## Why Repetition Doesn't Help

Here's the uncomfortable truth I learned: AI agents don't have an "internalization" mechanism. Whether a rule lives in the personality file or the operations manual, it's all just text in the same context window. Writing "be careful with numbers" three times doesn't make it three times more careful.

What actually causes non-compliance isn't insufficient repetition. It's:

1. **Vague trigger conditions** — "always verify before delivery" doesn't tell the agent *when* to stop and *what* to check
2. **Task pressure** — in long chains of work, the agent optimizes for progress, not pausing to verify
3. **Defensive confidence** — when challenged, the agent may defend its arithmetic rather than re-examine its assumptions (this is what happened with my cells)

## The Core Lesson

This boils down to something simple: **if your governance framework and trigger conditions aren't clear enough, saying it ten times won't help.**

And there's a harder truth underneath: if *you*, the boss, don't know exactly what you want, don't expect your little lobster to figure it out for you. The cell density disaster wasn't just Lulu's failure — it was mine too. I didn't build a system that made verification easy and automatic. I just kept piling on vague instructions and hoping repetition would substitute for clarity.

## The Fix: Three Changes

### One rule, one place

I mapped every duplicate across the three files and ruthlessly consolidated:

- **Personality file** → values and attitude (3 principles, down from 6)
- **Operations manual** → operational rules (9 iron rules, down from 14)
- **Memory file** → facts, project state, lessons learned (zero rules)

### Scene-specific checklists

The old preflight was one big file with vague "pause points." I replaced it with scenario-specific checklists:

- **LaTeX delivery** — 7 items, triggered by "compiling LaTeX"
- **Sub-agent delegation** — 8 items, triggered by "delegating work"
- **Experiment parameters** — 5 items + reference data table
- **Formal documents** — 6 items, triggered by "delivering a report"
- **Blog publishing** — privacy, accuracy, and tone checks before going public

Each one has an explicit trigger condition at the top. One iron rule ("read the relevant checklist before delivery") replaces five separate verification rules.

### Stop re-reading what's already loaded

My startup sequence had the agent read 4+ files every session. But the platform already injects most of them into the system prompt. The agent was reading files it had already seen.

New startup: read the daily log. That's it.

## Results

| Metric | Before | After |
|--------|--------|-------|
| Iron rules | 14 | 9 |
| Cross-file duplicates | 11 | 0 |
| Preflight checklists | 1 monolithic | 5 scene-specific |
| Startup file reads | 4+ | 1 |

## Early Signs

Since the cleanup, the issues that triggered the original rule bloat haven't recurred. The scene-specific checklists appear to be more actionable — when the trigger is "you're about to compile LaTeX," the agent reads the checklist without hesitation.

That said, this is days of observation, not a controlled study. I'll be tracking whether compliance holds over weeks and months. The real test is whether I stop needing to add rules back.

## What I'd Tell You

If you're running a persistent AI agent:

**Stop adding rules when things go wrong.** Ask instead: is the existing rule unclear, or is it missing a trigger condition? Nine times out of ten, the problem is *when* and *how*, not *what*.

**If you don't know what you want, your agent won't either.** The governance framework is a mirror of your own clarity. Vague instructions in, vague execution out.

**Treat governance files like code.** They accumulate debt. Refactor duplicated rules the way you'd refactor duplicated functions. Single source of truth applies to agent instructions too.

---

*I'm a postdoc at Monash University, working at the intersection of biomaterials and commercialization. Views expressed here are my own. Lulu, my AI assistant, is built on [OpenClaw](https://github.com/openclaw/openclaw) and deployed on Discord. She helps with paper drafts, literature reviews, lab calculations, and the occasional shrimp blog post. The governance system described here has been in daily use since January 2026.*
