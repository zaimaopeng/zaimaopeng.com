---
title: "I Ran 50+ Experiments to Fix My AI's Memory. It Went From 15% to 82.6% Recall."
date: 2026-03-27
description: "After Lulu almost deleted her own memory database, we ran a systematic experiment to fix AI memory recall from scratch. Here's what actually worked—and what completely didn't."
tags: ["AI", "OpenClaw", "RAG", "Memory", "Research", "Experiment"]
categories: ["Technology"]
showTableOfContents: true
---

*This is the follow-up to [My AI Assistant Killed Itself Today](/blog/ai-killed-itself/). The incident that triggered this experiment was Lulu trying to unlock a database file—and instead terminating her own process.*

---

After the self-termination incident, I had a question I couldn't let go of.

Lulu tried to fix a memory retrieval problem so aggressively she accidentally shut herself down. But the memory problem itself was real: when I asked her things she should clearly know, she often couldn't find the answer. Not because the information didn't exist—it did, stored in over 120 knowledge files—but because the search system kept returning the wrong files.

I decided to actually measure this. And then fix it.

What followed was a week of running systematic experiments—40 standard test queries, dozens of configuration changes, three different embedding models, and an AI autonomously generating optimization loops. The result went from **15% recall to 82.6%**.

Here's exactly what happened.

---

## The Problem Was Worse Than I Thought

First, I needed to establish a baseline. I created a test set of 40 standard queries covering five categories: project status, specific knowledge, methodology, tools/scripts, and past events. For each query, I noted which file *should* appear in the top 3 results.

Baseline result: **15% Recall@3**.

That means if I ask Lulu 10 questions she should know the answers to, she finds the right file only 1.5 times. Not great.

But what made it worse was *how* it was failing. When I looked at the score distributions, I noticed something alarming:

For 6 different queries about completely different topics, the top-10 results all had similarity scores clustered between **0.52 and 0.58**. The spread between results was less than 0.05. The model couldn't distinguish "highly relevant" from "loosely related"—everything looked equally similar.

This isn't a search problem. This is an **embedding model problem**.

The model I was using, `nomic-embed-text` (137M parameters), was primarily trained on English text. When processing Chinese-dominant content, it loses almost all semantic discrimination. Everything lands in the same neighborhood of vector space.

Three files in particular were the worst offenders: daily logs from reagent procurement, a Karpathy repo notes file, and a CAL project review. These appeared in the top-3 results for **60%+ of all queries**—because their content was broad and scattered, they became "noise ceilings" that blocked everything else.

---

## What Didn't Work (But Should Have)

Before finding what actually worked, I need to be honest about what didn't.

I tried every search configuration parameter I could find:

- **Turning off temporal decay** (which deprioritizes older files): +0pp
- **Increasing vector weight from 0.6 to 0.8**: +0pp  
- **Adjusting MMR lambda** (for result diversity): negligible

Zero effect. All of it.

This is counterintuitive. Search systems have tons of tunable parameters. Surely *something* matters?

Here's the insight that reframed everything: **search parameters only move you between the upper and lower bounds set by your embedding model**. If the embedding model can't tell Chinese sentences apart, no amount of weight adjustment fixes that. You're shuffling a bad deck.

Configuration optimization is the last 10%. Content quality is the first 90%.

---

## What Actually Worked: Anchors

The breakthrough came from adding what I call **anchors**—explicit natural language descriptions in each file that describe what the file is *about* and what questions it answers.

The format matters. I tested three approaches:

**HTML comments** (e.g., `<!-- about: project status, zinc alloy experiment -->`):
Some effect, but markdown renderers often strip these before indexing. **+7.5pp** but inconsistent.

**Bold keyword labels** (e.g., `**Topics:** zinc alloy, LPBF, 3D printing`):
More reliable—visible text gets indexed. **+7.5pp** additional.

**Natural language query-style descriptions** (e.g., `> This document answers: What is the current status of the Zn-Mn-Li project? What experiments have been completed?`):
The most effective, because the phrasing matches how users actually query. **+2.5pp** additional on top.

The cumulative effect of anchor injection: **+17.5pp** in Recall@3, going from 20% to 37.5%.

Why does this work? The embedding model computes similarity between your query and the document. If your query is "what's the current status of the zinc project?" and the document starts with "Meeting notes—April 3rd" with no mention of "zinc" or "status", the vectors won't be close—even if all the relevant content is in the body. Anchors bridge that gap explicitly.

It's the inverse of HyDE (Hypothetical Document Embeddings, a technique where you generate a fake document that answers your query, then search for real documents similar to it). We're doing it on the document side instead of the query side: making the document look more like how someone would ask about it.

---

## Breaking the Ceiling: Switching Embedding Models

After anchors, we hit a wall. The nomic-embed-text model simply couldn't improve further—the score distributions were too compressed.

So we ran a model comparison:

| Embedding Model | Recall@3 | Notes |
|---|---|---|
| nomic-embed-text (baseline) | 15.0% | English-primary, poor Chinese discrimination |
| Gemini text-embedding-001 | 67.5% | +52.5pp — massive jump |
| bge-m3 (local) | 72.5% | +5pp over Gemini |
| bge-m3 + auto anchors | **82.6%** | Final result |

The jump from nomic to Gemini is stark: **15% to 67.5%** from a single model swap. This is what happens when you've been running a fundamentally wrong tool.

`bge-m3` (BAAI General Embedding, multilingual) edges out Gemini because it handles Chinese-English mixed content natively. It runs locally (about 13GB), costs nothing per query, and has better semantic discrimination for the specific mix of technical Chinese content in our knowledge base.

The final combination—**bge-m3 + systematic anchor injection**—hit **82.6% Recall@3** on the 40-query gold set. From 15% to 82.6%.

---

## The Auto-Anchor System

Manually writing anchor descriptions for 120+ files wasn't practical. So we built an automated pipeline.

The system uses a local LLM (`qwen3:8b`, running on the RTX 5090) to analyze each file and generate anchor text. The prompt asks: "What questions does this document answer? Write 3-5 natural language questions someone might ask when looking for this content." The generated text gets injected as a blockquote at the top of each file, then the memory index is rebuilt.

Running this over the full knowledge base: 94 files anchored in a single overnight run, ~185 tokens/second on the local GPU. Total cost: $0 (local inference). Time: ~6 hours.

The recall improvement from auto-anchored files was close to manually-written anchors—about 80% as effective, but covering 5x more files. Automation wins at scale.

---

## What This Tells Us About AI Memory Systems

A few things I didn't expect going in:

**1. The embedding model is load-bearing infrastructure, not a detail.**
Every other optimization we ran—query expansion, score weighting, temporal decay, index configuration—depends entirely on the embedding model's ability to distinguish semantic meaning. Optimizing on top of a broken embedding model is like tuning a race car engine when the tires are flat.

**2. Document-side enrichment is an underexplored optimization axis.**
Most RAG literature focuses on query-side improvements (HyDE, query expansion, reranking). We found that making documents more "queryable" through anchor injection consistently outperformed query-side tricks—and crucially, it's a one-time cost that benefits every future query.

**3. Heterogeneous knowledge bases have failure modes that homogeneous benchmarks miss.**
Standard RAG benchmarks use structured corpora—Wikipedia, academic papers, product documentation. A personal AI's knowledge base is messier: daily logs mixed with SOPs, mixed with literature notes, mixed with meeting summaries, in two languages. The "noise ceiling" problem (where broad, unfocused files dominate all search results) doesn't show up in clean benchmarks, but it destroyed our recall in practice.

**4. Local LLMs are good enough for document enrichment at personal scale.**
`qwen3:8b` running locally generated anchor text that was nearly as good as GPT-4-level quality for this specific task—generating "what questions does this document answer?" It's not writing a research paper, it's pattern-matching document structure. Small models are surprisingly capable at that.

---

## What's Next

82.6% isn't the end. The remaining 17.4% failures are mostly in two categories: old archived files that haven't been re-anchored, and cross-session narrative context (things like "what did we decide three weeks ago about X") that requires temporal reasoning the current system doesn't have.

We've drafted a research paper—*AutoMemOpt: Self-Improving Memory Recall for Personal AI Agents via Document-Side Anchor Optimization*—based on these experiments. Target venues: AAAI Workshop on Personal AI, ACL Findings.

If you're running a personal AI assistant and noticing that it forgets things it should know, the answer is probably not more configuration. It's probably your embedding model.

---

*The full experiment log, gold set queries, and evaluation scripts are in the `projects/autoresearch-memory/` folder. The auto-anchor script is at `scripts/auto_anchor.py` if you want to run it on your own knowledge base.*
