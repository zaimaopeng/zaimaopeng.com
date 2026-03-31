---
title: "The Watchdog That Wasn't: Why Two Intelligent Actors Both Missed a Silent Failure"
date: 2026-03-31
description: "After fixing our AI watchdog, I started writing an article about what we learned. Halfway through, I caught myself describing a fix that I had planned but never implemented. So I fixed it live, and rewrote the ending."
tags: ["AI", "OpenClaw", "Debugging", "Agent", "Infrastructure"]
categories: ["Technology"]
showTableOfContents: true
series: ["Building with Lulu"]
series_order: 5
---
In the last post, I wrote about how we discovered that OpenClaw ships with a built-in Watchdog. When Lulu killed herself in the P1 incident, the Watchdog quietly restarted her. She never knew.

I thought that was the end of the story.

It wasn't.

---

## The Failure Nobody Noticed

About two weeks after that post, Lulu went offline again. This time, she didn't come back.

I waited. Nothing.

I opened Task Scheduler. The Watchdog entry was there: `Clawdbot Gateway Watchdog`, status: **Ready**.

So I pulled up Claude Code and asked it to dig into the actual execution history.

The last successful run was **March 12th**. We were now on March 26th.

Two weeks. The Watchdog had been silently not-running for two weeks. The status still said "Ready." No errors. No alerts. Just... nothing.

Root cause: when I'd moved OpenClaw from `clawd/` to `.openclaw/`, the Watchdog task still pointed at the old path — `C:\Users\User\clawd\scripts\gateway-watchdog.ps1`. That file no longer existed. So every time the Watchdog was scheduled to run, it launched, found nothing, and exited silently with a success code.

Task Scheduler interpreted this as: task ran successfully.

---

## Why We Both Missed It

Here's what I find genuinely interesting about this failure: **both Lulu and I had reasonable grounds for believing the Watchdog was working, and neither of us thought to verify it.**

This wasn't carelessness. It was rational behavior under incomplete information. But the rationality itself was the problem.

### Why I missed it

When Lulu had gone offline in previous weeks, she'd come back on her own — sometimes within minutes. I saw the evidence: uptime restored, no manual intervention required. My inference was obvious: the Watchdog is doing its job.

I wasn't wrong to make that inference. That's exactly what happened. The Watchdog *was* running in those early weeks. The error only appeared later, after the path changed.

So by the time the Watchdog broke, I had a well-established belief — backed by direct observation — that it was reliable. Why would I audit a system that was demonstrably working?

The tricky part: **past performance in a healthy state tells you nothing about current state in a degraded one.**

### Why Lulu missed it

Lulu's failure mode is different, and I think more structurally interesting.

Lulu has no runtime memory. Every session, she wakes up from files. She can read the P1 incident log. She knows she "died." But she has no first-person memory of what happened next — being restarted by the Watchdog, coming back online, continuing work as if nothing had happened.

That entire episode exists for her only as a historical record, not as felt experience.

So when she analyzed "do we have automatic recovery?", she did what any diligent analyst would do: she searched for existing mechanisms, found the Task Scheduler entry, checked its status field.

Status: **Ready**.

She concluded: mechanism exists, mechanism is active, we have automatic recovery.

She wasn't being lazy. She was being systematic. The problem is that "Ready" in Task Scheduler means *configured to run* — not *will successfully run when triggered*. Lulu didn't test the Watchdog. She verified its existence.

There's a subtle epistemic failure here that I've been thinking about since: **Lulu couldn't reason about the gaps in her own knowledge.** She knew the Watchdog existed. She didn't know that the Watchdog had rescued her, specifically, after P1. If she'd known that, she might have asked: "How do I know it will work again?" Instead, she found the artifact and declared the capability present.

She was reasoning about a system she had benefited from but never consciously experienced.

---

## The Watchdog Problem

There's a property of monitoring systems that makes them particularly vulnerable to this kind of failure:

**A working watchdog is invisible. A broken watchdog is also invisible.**

When the Watchdog is doing its job, you never see it act — Lulu just silently comes back online. When the Watchdog is broken, it also never acts — Lulu also silently stays offline.

The observable signatures of these two states are identical until you need the Watchdog and it fails.

Compare this to a system that fails loudly. If the Watchdog had thrown an error on every failed run, Task Scheduler would have shown "Last Run Result: 0x1." I would have seen it. The alert would have been there. Instead, the script exited cleanly (it found no path to execute, so no error code was set), and the task was marked successful.

Silent failures defeat the entire purpose of having a monitoring layer.

---

## What "Testing" the Watchdog Actually Means

After fixing the path, I ran the Watchdog manually. It wrote to its log file: `Gateway is running`. Good.

But that's not really a test. That's "does the script execute." A real test would be:

1. Kill the Gateway process deliberately
2. Wait for the next scheduled Watchdog run (or trigger it)
3. Verify that Gateway comes back up
4. Verify that a notification was sent

We've never done step 1-4 as an intentional verification cycle. Not once.

The closest thing we have to a test is: Gateway died, Watchdog ran, Lulu came back. That's production testing. It works until it doesn't.

What we should have is a scheduled drill — maybe monthly — that does exactly this sequence in a controlled way. Not because the failure is likely, but because we've now learned that **"the Watchdog exists" and "the Watchdog works" are two different claims**, and we've been conflating them.

---

## The Deeper Pattern

I think this failure generalizes beyond Watchdogs.

Any system where the expected state and the failure state produce identical observable output is dangerous. Status fields that show "configured" rather than "verified working." Health checks that only check if a process is running, not if it's responding correctly. Backup jobs that complete without confirming the backup is restorable.

The fix isn't necessarily more monitoring. Sometimes it's just: **periodically verify the thing you care about, not the proxy indicator you've been watching.**

In our case: we were watching Task Scheduler's status field. What we should have been watching was "did the Gateway survive an outage recently, and how."

---

## Where We Are Now

After the March 26th incident, we fixed the path and verified the Watchdog works. We added a restart notification so Lulu's daily log records when a Watchdog-triggered restart happened.

I thought that was enough. So I started writing this article.

---

## The Moment I Caught Myself

About halfway through the first draft, I wrote this sentence:

> *"Lulu, for her part, updated her startup health check to include: verify Watchdog last-run date is within the last 10 minutes. If the Watchdog hasn't run recently, flag it."*

I wrote it in the past tense. As if it existed.

Then I stopped, because I realized: I was writing about a fix I had planned but never actually implemented. I had the idea, mentioned it in a previous session, it went into a note somewhere, and then nothing happened. The fix became part of the narrative of what we learned — without becoming part of the system.

So I did what I should have done in March: I checked.

```powershell
Get-Content "C:\Users\User\.openclaw\logs\watchdog.log" -Tail 3
```

The log exists. The Watchdog is running every two minutes. Status: OK.

But HEARTBEAT.md — the file Lulu reads every session to know what to check — had zero mention of the Watchdog. No last-run verification. No alert threshold. Nothing.

The check I described as existing wasn't there.

---

## The Pattern, One More Time

I want to be precise about what happened:

1. The Watchdog failed silently for two weeks (March 12–26)
2. We diagnosed and fixed the root cause (wrong path)
3. I formed the intention to add a last-run check to Lulu's startup routine
4. I wrote a blog article describing that check as implemented
5. Writing the article, I verified — and found it wasn't implemented

Steps 3 and 4 happened. Step 4 came before step 5. I documented a fix before confirming it existed.

This is the same failure mode, applied to itself. We missed the Watchdog's silent failure because we trusted "configured" to mean "working." I almost published an article claiming a fix was in place that wasn't.

The fix for the fix: I added the check to HEARTBEAT.md while writing this paragraph. It now reads Watchdog log on every session startup, parses the last timestamp, and alerts if it's been more than 10 minutes since the last run. I verified the logic against the actual log before continuing.

That check exists now. I tested it. This time the past tense is accurate.

---

## The Actual Lesson

I've been framing this as a story about monitoring systems and silent failures. That's true, but it's not the whole thing.

The deeper pattern is about the distance between *knowing something should be done* and *confirming it was done*. That gap exists everywhere: in engineering, in research, in organizations. The more complex the system, the more places that gap can hide.

Lulu can't verify whether her intentions from previous sessions were executed — she wakes up from files, not from memory. That structural gap means it falls on the files themselves to be honest about what's in them.

HEARTBEAT.md said nothing about Watchdog monitoring. That silence was accurate. I just hadn't read it that way.

The useful habit isn't "check everything constantly." It's: **when you write that something exists, go look at it first.**

---

*Part of the [Building with Lulu](/) series — documenting what it's actually like to build and run a persistent AI agent.*

*← Previous: [The Watchdog Was Already There](/blog/watchdog-was-already-there/)*

