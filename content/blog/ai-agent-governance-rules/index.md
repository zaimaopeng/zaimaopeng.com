---
title: "I Had 14 Rules for My AI Agent. Most Were Useless."
date: 2026-03-19
description: "How I went from governance bloat to a system that actually works — lessons from two months of running a persistent AI assistant."
tags: ["AI", "Workflow", "Productivity", "Agent"]
categories: ["Technology"]
showTableOfContents: true
---

Two months ago, I set up an AI assistant that runs 24/7 — reading my files, managing my research projects, helping me write papers. It has three core files that define who it is and how it works: a personality file, an operations manual, and a long-term memory file.

Over time, I kept adding rules. Every mistake got a new rule. Every near-miss got a safeguard. By March 2026, I had **14 iron rules**, a monolithic preflight checklist, and the same instruction written in three different files.

The agent wasn't following most of them.

## The Incident That Started the Rule Bloat

In mid-March, my AI gave me cell seeding densities that were off by an order of magnitude — 10⁴ instead of 10⁵. I didn't catch it in time. An entire batch of cells went to waste.

My response? Add a rule: "Cross-validate experimental parameters against known reference values." Reasonable.

But then I added the same rule to the personality file ("feel the weight of unsupported claims"), restated it in the memory file ("experimental parameters must be cross-validated"), and wrote a dedicated checklist section.

**The same rule, written four times, in four files. And the agent still didn't reliably follow it.**

## Why Repetition Doesn't Help

Here's what I learned by asking my agent directly: it doesn't have an "internalization" mechanism. Whether a rule lives in the personality file or the operations manual, it's all just text in the same context window. Writing "be careful with numbers" three times doesn't make it three times more careful.

What actually causes non-compliance isn't insufficient repetition. It's:

1. **Vague trigger conditions** — "always verify before delivery" doesn't tell the agent *when* to stop and *what* to check
2. **Task pressure** — in long chains of work, the agent optimizes for progress, not pausing to verify
3. **Rule overload** — 14 rules means none of them stand out

## The Fix: Three Changes

### One rule, one place

I mapped every duplicate across the three files and ruthlessly consolidated. Each rule now lives in exactly one location:

- **Personality file** → values and attitude (3 principles, down from 6)
- **Operations manual** → operational rules (9 iron rules, down from 14)
- **Memory file** → facts, project state, lessons learned (zero rules)

The rule about experimental validation? It's in the operations manual, once. The cell incident is in the memory file as a lesson — not a rule, but a story that explains *why* the rule exists.

### Scene-specific checklists

The old preflight was one big file with vague "pause points." I replaced it with four scenario-specific checklists:

- **LaTeX delivery** — 7 items, triggered by "compiling LaTeX"
- **Sub-agent delegation** — 8 items, triggered by "delegating work"
- **Experiment parameters** — 5 items + reference data table
- **Formal documents** — 6 items, triggered by "delivering a report"

Each one has an explicit trigger condition at the top. The agent doesn't need to decide *whether* to check — if the scenario matches, it reads the file. One iron rule ("read the relevant checklist") replaces five separate verification rules.

### Stop re-reading what's already loaded

My startup sequence had the agent read 4+ files every session. But the platform already injects most of them into the system prompt automatically. The agent was literally reading files it had already seen.

New startup: read the daily log. That's it.

## Results

| Metric | Before | After |
|--------|--------|-------|
| Iron rules | 14 | 9 |
| Cross-file duplicates | 11 | 0 |
| Preflight checklists | 1 monolithic | 4 scene-specific |
| Startup file reads | 4+ | 1 |

## Early Signs

Since the cleanup, the issues that triggered the original rule bloat haven't recurred. The scene-specific checklists appear to be more actionable — when the trigger is "you're about to compile LaTeX," the agent reads the LaTeX checklist without hesitation. The old monolithic "check everything" approach left too much room for "I'll check later."

That said, this is days of observation, not a controlled study. I'll be tracking whether compliance holds over weeks and months, and whether the reduced rule count introduces any new failure modes. The real test is whether I stop needing to add rules back.

## What I'd Tell You

If you're building a persistent AI agent with governance files:

**Stop adding rules when things go wrong.** Instead, ask: is the existing rule unclear, or is it missing a trigger condition? Nine times out of ten, the problem is *when* and *how*, not *what*.

**Treat governance files like code.** They accumulate debt. The same way you'd refactor duplicated functions, refactor duplicated rules. Single source of truth applies to agent instructions too.

**Your files serve two audiences.** The AI reads them as flat text. You maintain them as structured documents. Optimize for both — but when they conflict, optimize for human maintainability, because you're the one who'll create contradictions.

---

*I'm a postdoc at Monash University, working at the intersection of biomaterials and commercialization. Views expressed here are my own and do not represent the university. My AI assistant runs on [OpenClaw](https://github.com/openclaw/openclaw) and helps with everything from literature reviews to lab calculations. The governance system described here has been in daily use since January 2026.*
