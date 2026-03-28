---
title: "I Hired Another Claude to Grade My AI. It Scored 4.3/5 and Found 3 Blind Spots I Missed."
date: 2026-03-28
description: "My AI assistant can rewrite its own rules. That's a trust problem. So I built a system where a second Claude evaluates every change—and coaches the first one after complex tasks. Here's how it works and what happened on day one."
tags: ["AI", "OpenClaw", "Governance", "Claude Code", "BuildInPublic"]
categories: ["Technology"]
showTableOfContents: true
---

*This is part of a series about building a personal AI assistant. Previously: [governance rules](/blog/ai-agent-governance-rules/), [the self-termination incident](/blog/ai-killed-itself/), [fixing memory recall](/blog/memory-from-15-to-82/).*

---

## The Problem: Who Watches the Watchman?

My AI assistant Lulu runs on a governance framework—a set of "iron rules" that define what she can and can't do. She has a soul file that shapes her personality. A memory routing table that determines how she finds information. Configuration files that control her daily behaviors.

She also has write access to all of these files.

This means the entity being governed is the same entity maintaining the governance system. She writes her own rules, evaluates her own performance, and decides when rules need changing.

For two months, this worked fine. Lulu would propose a change, I'd review it, and we'd move on. But as the system grew more complex—11 iron rules, 6 departments, 25 skills, 15 cron jobs—I couldn't review every governance tweak in real time. Some changes were subtle: a word here, a threshold there, a new exception buried in a config file.

I needed a second pair of eyes. Specifically, I needed someone who could read Lulu's entire workspace, understand the governance framework, and evaluate proposed changes independently.

I needed another Claude.

---

## The Inspiration

Anthropic published a design pattern for what they call a "harness" architecture—a three-agent system where one agent does work, another evaluates the work, and a human makes final decisions. The key insight: the evaluator and the worker should be separate instances with separate contexts.

I adapted this into a dual-role system I call the **CC Protocol** (Claude Code Protocol). "CC" is Claude Code, running locally on my terminal—completely separate from Lulu's OpenClaw instance.

CC has two roles:

| Role | When | Blocking? |
|------|------|-----------|
| 🔒 **Guardian** | Before any governance file change | Yes — change doesn't happen until CC approves |
| 🏋️ **Coach** | After complex tasks are completed | No — Lulu finishes work first, CC reviews async |

Same Claude model, same capabilities, but running in a different context with different instructions. The guardian sees proposals before they take effect. The coach sees retrospectives after work is done.

---

## How the Guardian Works

When Lulu wants to modify a governance file—AGENTS.md, MEMORY.md, SOUL.md, or any iron rule—she can't just edit it. Instead:

1. She writes a **proposal** to `handoff/inbox/` with a structured format: what she wants to change, why, a diff, and a self-assessment of risk
2. A poller script (running every 30 minutes) detects the new file
3. Claude Code reads the proposal along with Lulu's entire workspace
4. CC produces an evaluation: PASS, REVISE, or FAIL, with a score out of 10 and specific feedback
5. If PASS: Lulu applies the change. If REVISE: she adjusts and resubmits (up to 3 rounds). If FAIL: the change is blocked

The state machine:

```
idle → awaiting-eval → awaiting-revision → idle
              ↑                    |
              +--------------------+  (up to 3 revision rounds)
```

The critical constraint: **Lulu cannot modify governance files without going through this gate.** This is iron rule #3 in her constitution-level rules. She enforces it on herself, and the file-based protocol makes it auditable.

---

## How the Coach Works

The guardian blocks bad changes. The coach improves good work.

After Lulu completes a complex task—a major refactor, a system optimization, a multi-step project—she writes a retrospective and drops it in the same `handoff/inbox/` directory, tagged as `type: retrospective`.

CC reads the retrospective and scores it on five dimensions:

| Dimension | What It Measures |
|-----------|-----------------|
| Planning | Was the approach structured? Were risks identified? |
| Execution | Clean path or lots of rework? |
| Self-Review Honesty | Did Lulu admit failures and root causes? |
| Risk Awareness | Edge cases and second-order effects? |
| Knowledge Capture | Were lessons written to the right files? |

Each dimension gets a 1-5 score. CC also identifies blind spots—things Lulu didn't notice or consider—and gives concrete improvement suggestions.

The key difference from the guardian: **the coach doesn't block anything.** Lulu completes the work, writes the retrospective, and moves on. CC's feedback arrives asynchronously and gets picked up in the next session.

---

## Day One: The First Real Test

On March 28, Lulu ran a skill architecture audit. She reviewed all 28 skills in her system, identified overlaps, and consolidated:

- Deleted 2 redundant skills
- Merged 1 pair of overlapping skills (citation network tools)
- Updated the memory routing table
- Verified token budget stayed within limits

After completing the work, she wrote a retrospective and submitted it to the coach.

**CC's verdict: 4.3/5.**

The dimension breakdown:

| Dimension | Score |
|-----------|-------|
| Problem Identification | 4/5 |
| Data Support | 5/5 |
| Improvement Direction | 4/5 |

Not bad for a first submission. But the interesting part was the blind spots.

---

## The 3 Blind Spots CC Found

### 1. No Recurring Audit Schedule

Lulu audited skills because I asked her to. CC pointed out: there was no mechanism to detect skill decay over time. A skill that hasn't been triggered in 30 days might be dead weight—consuming token budget without providing value.

**CC's suggestion:** Monthly automated audit. Check last-used dates. Archive anything dormant for 30+ days.

This was a genuine gap. Lulu had optimized the current state but hadn't designed for ongoing maintenance.

### 2. No Standardized Skill Template

Each skill's documentation (SKILL.md) had a different format. Some listed "when to use" triggers. Some didn't. Some had "when NOT to use" guidance. Most didn't.

**CC's suggestion:** Minimum template: name / when-to-use / when-NOT-to-use / entry-point.

Again, a real issue. The inconsistency meant Lulu sometimes loaded the wrong skill because the trigger descriptions were vague.

### 3. Review Granularity Too Coarse

This audit covered 28 skills accumulated over 60 days. That's a lot of surface area for one review.

**CC's suggestion:** Lighter reviews every two weeks, focusing only on new and changed skills. Reserve the full audit for monthly cadence.

Fair point. Smaller, more frequent reviews catch drift earlier.

---

## The Information Asymmetry Bug

Here's where it got interesting.

After reading CC's feedback, I noticed something: some of CC's suggestions were about things Lulu had *already done*. CC recommended consolidating citation network tools—but Lulu had already merged them. CC suggested cleaning up workspace-literature-review—already deleted.

Why? Because Lulu's retrospective described what she *planned and analyzed*, but not what she *actually executed*. CC only saw the retrospective text. It didn't know the operations had been completed.

This was a protocol design flaw. The retrospective template required:
- What was planned
- What was the result
- Self-assessment

But not:
- What files were actually created, deleted, or modified

So I fixed it on the spot. The retrospective template now requires an **execution log**:

```markdown
## Execution Log
- Deleted: [file path]
- Created: [file path]
- Modified: [file path] — [what changed]
```

And the coach prompt now includes an instruction to **verify claimed operations** by checking the actual file system—`ls` and `cat` the files to confirm they exist (or don't).

The meta-lesson: a system where one agent reviews another only works if the information transfer is complete. Partial retrospectives produce partial evaluations. The protocol was correct in structure but lossy in data.

---

## The Architecture

Here's the full picture of how governance works now:

```
Lulu (OpenClaw)                    CC (Claude Code)
     |                                    |
     |-- writes proposal.md ------------>|
     |                                    |-- reads workspace
     |                                    |-- evaluates
     |<-- eval-report.md ----------------|
     |                                    |
     |-- applies (if PASS)               |
     |-- revises (if REVISE, up to 3x)   |
     |                                    |
     |== after task completion ===========|
     |                                    |
     |-- writes retrospective.md ------->|
     |                                    |-- scores 5 dimensions
     |                                    |-- identifies blind spots
     |<-- coach-review.md ---------------|
     |                                    |
     |-- picks up in next session         |
```

The two Claudes never talk directly. All communication is through markdown files in a shared directory. A PowerShell poller script checks every 30 minutes, and Lulu can also trigger evaluation immediately via CLI.

Michael (the human) remains the final authority. CC can block a governance change, but Michael can override. CC's coaching scores are informational, not binding.

---

## What I Learned

**1. Separation of concerns applies to AI governance too.**

The same entity shouldn't write rules, follow rules, and evaluate compliance with rules. Even if the "entity" is an AI that can theoretically be objective. In practice, self-evaluation has structural blind spots—you can't see what you don't think to look for.

**2. The coach role is more valuable than the guardian role.**

Blocking bad changes is table stakes. But the coach catches *patterns*—recurring weaknesses, missing habits, systematic gaps. Over time, coaching should compound: each review makes the next task execution slightly better.

I expect the guardian to fire rarely (most governance changes are low-risk). I expect the coach to produce insight every time.

**3. File-based protocols are underrated.**

No API calls between agents. No complex orchestration framework. Just markdown files in a directory, a polling script, and clear naming conventions. It's debuggable (read the files), auditable (check the archive), and resilient (if the poller crashes, nothing is lost—just delayed).

**4. Day-one bugs are the best bugs.**

The information asymmetry issue appeared on the very first real use. If I'd waited a month to test, the retrospective template flaw would have silently degraded every coach review in between. Ship early, break early, fix early.

---

## Current State

The CC Protocol has been running for one day. The archive contains one completed coaching cycle. The guardian hasn't been tested yet (no governance changes since deployment).

Upcoming:
- First guardian test: I'm planning a change to the iron rules that will trigger the full proposal → evaluation → approval flow
- Pattern detection: Once the coach has 5+ reviews, CC should start identifying recurring issues across retrospectives
- Scoring trends: I want to track whether Lulu's scores improve over time—the whole point of coaching

The system is simple, file-based, and completely transparent. Every proposal, every evaluation, every score lives in a markdown file that anyone can read.

That's the whole point. When your AI can rewrite its own rules, the process for doing so should be the most visible thing in the system.

---

*Next in the series: I've been running 15 cron jobs that automate everything from morning briefings to overnight paper discovery. Here's what a fully automated AI assistant schedule looks like—and the jobs that keep failing.*
