---
title: "When Good Notes Stop Being Searchable: Why I Built an Anchor System for My AI Memory"
date: 2026-04-13
description: "As my AI-assisted workspace grew, I realized that having good notes was no longer enough. The real problem was retrieval. This is the story of why I built an anchor system, why the first version broke, and how we rebuilt it into a resumable production pipeline."
tags: ["OpenClaw", "AI", "Ollama", "Memory", "Search", "Infrastructure", "Automation"]
categories: ["Technology"]
showTableOfContents: true
---

# When Good Notes Stop Being Searchable: Why I Built an Anchor System for My AI Memory

There is a strange phase that happens when your personal knowledge system starts getting genuinely useful.

At the beginning, every new note feels like progress. Every daily log, every research memo, every department document, every technical lesson makes the system richer. You feel like you are building a second brain.

Then one day, almost without noticing, the problem changes.

The problem is no longer **whether the knowledge exists**.
The problem is **whether you can reliably get it back at the moment you need it**.

That was the point where I started thinking seriously about anchors.

I was already using OpenClaw heavily across writing, memory, projects, and internal knowledge organization. The workspace was getting richer, which was good. But richer systems also become fuzzier systems. A lot of the information I wanted *was* in the workspace somewhere — in a daily log, in a department file, in a draft, in a technical protocol — but retrieval was starting to depend too much on whether my future query happened to match the wording of the original document.

That mismatch bothered me.

Because when humans look for old knowledge, we rarely search using the exact language that existed when we wrote it. We search using the language of our current problem.

That is the real origin of this anchor system.

I did not start with “let’s optimize embeddings.” I started with a much simpler frustration:

> I knew the knowledge was in the system. I just did not trust that the system would always find it the way a human would ask for it later.

This post is the story of what the anchor system is, why it mattered, why the first version broke, and how we rebuilt it into a resumable production pipeline.

## What the anchor system is actually for

Our workspace contains a large and growing collection of Markdown files: daily logs, memory files, department knowledge bases, technical notes, drafts, and long-form writing.

The problem is that raw semantic search is often not enough.

A file may be highly relevant to a future query even if the exact words a user will type do not appear in the document. A person might ask:

- “Why is deep work still important in the AI era?”
- “How did we structure external evaluation for governance changes?”
- “Did ancient China really have something like book clothing?”

But the actual document may phrase the same idea differently.

That is where anchors come in.

For each document, we generate a short block of hypothetical search queries — natural-language questions a future user might ask when trying to retrieve that file. We then inject that block into the document as a lightweight retrieval aid.

In practice, the injected block looks like this:

```md
<!-- auto-anchor-v1 -->
> **Hypothetical queries:** query A | query B | query C
<!-- auto-anchor-v1 -->
```

Conceptually, this is a kind of reverse HyDE: instead of generating hypothetical documents at query time, we generate hypothetical queries at indexing time.

## Why we wanted to automate it

Manually writing search anchors does not scale.

As the knowledge base grows, the cost of maintaining retrieval quality grows with it. If we want the memory system to stay useful across many days, projects, and departments, anchor generation has to become automatic.

Our initial design was straightforward:

1. Scan a target folder such as `departments/` or `memory/`
2. For each Markdown file, send the first part of the document to a local LLM
3. Ask the model to generate 5–8 plausible retrieval queries
4. Inject the result back into the file
5. Re-index the memory system

On paper, it was a clean pipeline.

In reality, it was much more fragile than it looked.

## What went wrong in the first version

The first version of the script worked only as long as everything went right.

That is fine for a quick experiment. It is not fine for production.

We ran into three failure modes.

### 1. Terminal encoding issues on Windows

The script printed Unicode symbols in progress logs. On Windows, that caused repeated errors such as:

```text
'charmap' codec can't encode character '\u2192'
```

The actual anchor generation logic was sometimes fine, but the job still failed because the logging layer crashed.

This is a classic lesson in automation work: operational robustness is not just about the model call. Your logs, shell, encoding assumptions, and I/O path are all part of the system.

### 2. Local model calls occasionally timed out

We use a local Ollama-served model for anchor generation. That choice is attractive because it is cheap, private, and easy to run. But local inference still has real latency and variability.

Some files returned quickly. Others stalled long enough to trigger request timeouts.

That meant a single slow file could block a long-running batch job.

### 3. One long process was doing too much

The biggest architectural problem was this: one Python process was responsible for the whole run.

So when a single file stalled or the host killed the long-running task, the entire batch went down with it.

Even worse, progress was not always safely persisted after each file. That meant the system could succeed on several items and still leave us with the feeling that “nothing worked,” because the top-level process died before the run ended cleanly.

That is the difference between a script that can sometimes work and a system that can be trusted.

## The design goal for the rebuild

Once the failure pattern became clear, the right direction was obvious.

We did **not** need a smarter prompt first.
We needed a more fault-tolerant execution model.

The design goal became:

> A single file should be allowed to fail, time out, or retry without taking down the entire anchor generation run.

That is what pushed us from a script mindset into a pipeline mindset.

## The rebuild: from batch script to three-stage production pipeline

We rebuilt the anchor system into three explicit stages:

### 1. Planner

The planner scans the target directory and decides which files are eligible for processing.

It considers things like:

- whether the file is too short to be worth anchoring
- whether an anchor already exists
- whether the file has already been processed successfully
- whether the content hash has changed since the last run
- whether previously failed files should be retried

Instead of immediately processing files, the planner writes a queue.

That queue becomes the handoff point between discovery and execution.

### 2. Worker

The worker handles exactly **one file**.

It reads the file, calls the local model, generates hypothetical queries, injects the anchor, and returns a structured result payload.

The key difference is isolation.

A worker is disposable. If it times out, crashes, or gets killed, we lose only one file attempt — not the whole batch.

### 3. Runner

The runner consumes the planner’s queue one file at a time by launching workers as subprocesses.

After each file, it immediately writes:

- a durable `state` record
- a durable `results` record
- status metadata such as success, timeout, attempts, and duration

This means the system becomes resumable by design.

If the host kills the job halfway through, we do not lose the entire run. We just continue from the last known state.

## What changed technically

The upgraded version now includes the following production features:

- **planner / worker / runner architecture**
- **per-file subprocess isolation**
- **state persistence** in `auto_anchor_state.json`
- **result persistence** in `auto_anchors.json`
- **resume support**
- **retry support** for timed-out or failed files
- **content hashing** so changed files can be reprocessed
- **safer UTF-8 output handling** on Windows
- **batch limiting** for controlled rollout and testing

The practical effect is simple: we moved from “one fragile long job” to “many small accountable jobs.”

## What the first production run showed

The upgraded pipeline was then tested on `departments/`.

A representative run processed three queued files:

- `departments/content/production/book-clothing-ancient-china/blog_draft_en.md`
- `departments/content/production/book-clothing-ancient-china/blog_draft_zh.md`
- `departments/content/production/deep-work-is-not-obsolete-in-the-ai-era/index.zh-cn.md`

The interesting part was not just that these files succeeded.

The most important proof point was that one of them **timed out on the first attempt and then succeeded on retry**, while the overall run still completed successfully.

That is exactly the behavior we wanted from the redesign.

The final run summary showed:

- success: 3
- timeout: 0
- failed: 0

And after that, a full planner+runner pass over `departments/` queued **0 additional files**, meaning the remaining eligible items under the current rules had already been processed.

## Is the problem “solved”? My honest view

Yes — but with an important nuance.

If the question is:

> “Is the original anchor-system problem solved well enough to call this a working production upgrade?”

Then my answer is **yes**.

The key problem was not “can the model generate anchors?” It clearly could.
The real problem was “can the system survive imperfect real-world execution?”

Now it can:

- a single file can fail without collapsing the batch
- retries work
- progress is persisted
- runs are resumable
- successful work is not lost when one item struggles

That is the threshold that matters.

If the question is:

> “Is this the final possible form of the system?”

Then of course not.

There are still useful next steps:

- richer metrics and dashboards
- better model routing for different document types
- selective incremental runs based on recent file changes
- automatic post-run memory indexing
- periodic scheduled execution with alerts

But those are optimization steps, not existential repairs.

The system has crossed the line from experimental fragility to operational usefulness.

## The broader lesson

This upgrade was a reminder of a pattern that shows up in many AI systems.

People often think the hardest part is prompt design or model quality. Sometimes it is. But very often, the real difference between a demo and a reliable tool is much more mundane:

- checkpointing
- retries
- state persistence
- failure isolation
- resumability

In other words, the path to better AI systems often looks suspiciously like ordinary systems engineering.

That is not disappointing. It is actually encouraging.

Because once the core model capability is good enough, reliability becomes an engineering problem — and engineering problems can be systematically solved.

## Where this goes next

With the anchor generation pipeline stabilized, the next step is not to keep rewriting the same script forever.

The next step is to treat anchor generation as a first-class part of the memory maintenance pipeline:

- detect new or changed knowledge files
- generate anchors incrementally
- persist state and metrics
- re-index only when appropriate
- surface meaningful summaries rather than raw logs

That is how you stop a memory system from quietly degrading as the workspace grows.

And that, ultimately, is what this upgrade was really about.

Not just adding anchors.

Building a retrieval system we can trust.
