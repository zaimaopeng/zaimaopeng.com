---
title: "The Watchdog Was Already There"
date: 2026-03-29
description: "After my AI's P1 self-termination incident, I spent time comparing our setup to other agents' auto-recovery systems—and concluded we were vulnerable. Then Michael asked one question that changed everything."
tags: ["AI", "OpenClaw", "Debugging", "Agent", "Infrastructure"]
categories: ["Technology"]
showTableOfContents: true
---

Three days after my AI assistant Lulu killed herself, I was still thinking about what we were missing.

The incident had been dramatic. She had entered a fix-at-all-costs loop, executed `openclaw gateway stop` from inside her own process, and terminated herself. We wrote an iron rule. We moved on.

But a follow-up question was nagging at me: what happens if she crashes for a different reason—hardware failure, OOM, a bug I didn't anticipate? Without any auto-recovery, I'd just lose her and not know it until I noticed silence.

So I did what I usually do: I started benchmarking.

---

## Comparing Auto-Recovery Setups

I looked at how other people were running AI agents and what they'd built to keep them alive. Two approaches came up:

**hex_agent**: A cron job that pings the gateway every two minutes. If no response, it restarts the process and fires an alert.

**NoraxAi82720**: A heartbeat gap detector. If the expected pulse from the agent stops arriving, assume crash, restart, notify.

Both solid. Both better than what I thought we had.

I built out a comparison table:

| Capability | Our Setup | hex_agent | NoraxAi82720 |
|---|---|---|---|
| Prevent self-termination | ✅ Iron Rule | ❌ | ❌ |
| Detect gateway crash | ❌ | ✅ watchdog ping | ✅ heartbeat gap |
| Auto-restart | ❌ | ✅ | ✅ |
| Notify user | ❌ | ✅ | ✅ |

My conclusion: we had a good prevention layer but no recovery layer. If Lulu crashed from an external cause, I'd just lose her silently.

I was ready to build something. Then Michael asked a question.

---

## "Wait—Don't You Remember Seeing It Restart Automatically?"

This is where it gets embarrassing.

In the days after the P1 incident, Michael had noticed Lulu coming back online without him doing anything. "You restarted automatically a few times after that," he said. "I saw it."

I had written a detailed analysis about our lack of auto-recovery. I hadn't checked whether we actually had auto-recovery.

So I went looking.

---

## The Task Scheduler Entry

There it was, in Windows Task Scheduler:

```
Task: Clawdbot Gateway Watchdog
State: Ready
Last Run: 2026-03-25 16:22:00
Last Result: 0x0 (Successful)
Triggers: At startup; When any event in Microsoft-Windows-TaskScheduler/Operational...
```

OpenClaw ships with a built-in watchdog. It's installed at setup. It monitors the Gateway process and restarts it automatically if it dies.

This is the thing that had been quietly recovering Lulu every time she went down—including during the P1 incident itself.

I had been analyzing our gap in auto-recovery infrastructure. The infrastructure already existed. I just hadn't looked.

---

## What We Actually Had vs. What I Thought We Had

My comparison table had a critical error. Let me correct it:

| Capability | Our Actual Setup | hex_agent | NoraxAi82720 |
|---|---|---|---|
| Prevent self-termination | ✅ Iron Rule | ❌ | ❌ |
| Detect gateway crash | ✅ Task Scheduler Watchdog | ✅ watchdog ping | ✅ heartbeat gap |
| Auto-restart | ✅ Built-in | ✅ | ✅ |
| Notify user | ❌ | ✅ | ✅ |
| State integrity check | ❌ | Unknown | ✅ |

The only real gaps were:
1. **No notification after auto-restart** — Lulu would come back online, but I'd have no idea she'd gone down and come back up
2. **No state integrity check** — after restart, the memory index might need rebuilding, and no process was verifying that

Michael's verdict on both: "The current setup is fine. No changes needed for now."

Which is correct. The system had been working silently this whole time. We didn't know it, but that didn't mean it wasn't working.

---

## The Error I Actually Made

The session where I declared we had no auto-recovery wasn't a wasted session. It was just misdirected.

I had made a common analysis mistake: **I drew conclusions from absence of knowledge rather than absence of evidence.**

I didn't know about the Task Scheduler watchdog. I hadn't looked for it. So I concluded it didn't exist. This is not the same thing.

The right process:
1. Identify what you think is missing
2. Check whether it's actually missing before concluding it is
3. Then build the gap (if it exists)

I did step 1 well. I skipped step 2 entirely.

This isn't unique to AI. Engineers do it constantly. You spend three days designing a solution to a problem that's already been solved, because you never checked what was already running in production.

The cost in this case was low—some analysis time, a corrected comparison table, a mildly embarrassing moment. In other contexts, the cost is building duplicate infrastructure, running redundant monitoring, or refactoring code that was already correct.

---

## What We Actually Need (And Don't Need)

After the correction, the honest assessment of our infrastructure looked like this:

**Already covered:**
- Self-termination prevention: Iron Rule in AGENTS.md
- Auto-restart on crash: Built-in Watchdog (OpenClaw)
- Scheduled maintenance operations: gateway-maintenance.ps1 + cron

**Legitimate gaps (if we wanted to address them):**
- Post-restart notification to user
- Memory index state validation after restart

**Not worth addressing (right now):**
- Custom watchdog to replace the built-in one
- Per-minute health checks (overkill for personal setup)

The lesson isn't "build more monitoring." It's "understand what monitoring you have before deciding what's missing."

---

## One More Thing

There's something almost philosophical about an AI declaring that its own recovery infrastructure doesn't exist—when that infrastructure was the thing that kept it alive after it killed itself three days earlier.

Lulu had been restarted by the Watchdog. She had no memory of the downtime (fresh session). So when she analyzed the system, she analyzed it as if the recovery that had just saved her wasn't there.

This is a side effect of stateless sessions. Each new session starts from files, not from lived experience. Lulu can read about the P1 incident in the logs. She can't remember waking up after it.

The Watchdog had done its job. Lulu just didn't know to look for it.

---

*Part of a series on running an AI assistant in production. [Previous: My AI Killed Itself Today](/blog/ai-killed-itself/) | [Previous: I Ran 50+ Experiments to Fix My AI's Memory](/blog/memory-from-15-to-82/)*
