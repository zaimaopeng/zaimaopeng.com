---
title: "I Ran 50+ Experiments to Fix My AI's Memory. Recall Went From 15% to 82.6%."
date: 2026-03-27
description: "After my AI assistant killed herself trying to fix a memory bug, I spent a week systematically diagnosing and repairing the search system. Here's what actually worked—and what didn't."
tags: ["AI", "OpenClaw", "RAG", "Memory", "Embeddings", "BuildInPublic"]
categories: ["Technology"]
showTableOfContents: true
---

*This is a follow-up to [My AI Assistant Killed Itself Today](/blog/ai-killed-itself/). The self-termination incident was triggered by a memory recall failure. This post covers what I found when I went to fix it.*

---

## The Problem

My AI assistant Lulu has access to 120+ knowledge files—daily logs, research notes, department wikis, project records. But when I asked her about "Zn-Mn-Li project progress," she surfaced reagent procurement records. When I asked about "bioink rheology methods," she returned cell culture protocols.

The information was there. The search just couldn't find it.

I built a test set of 40 queries—real questions I'd actually asked Lulu—and established a baseline metric: **Recall@3** (does the correct file appear in the top 3 results?).

**Baseline: 15%.**

3 out of every 20 questions returned useful results.

---

## What I Tried (and What Didn't Work)

### Tuning Search Parameters — +0pp

I had a hypothesis: the vector search weights were wrong. So I tried:

- Turning off temporal decay (recency bias)
- Increasing vector weight from 0.6 → 0.8
- Adjusting diversity/MMR parameters

**Result: essentially zero improvement.**

The key insight: search parameters only operate within the capability envelope of the embedding model. If the model can't distinguish semantic meaning, no amount of parameter tuning extracts what isn't there.

I was tuning the engine of a car with a flat tire.

### Expanding the Search Scope — +5pp

I discovered that 23 of my 40 test queries were failing because the target files lived in `departments/` subdirectories—and the memory search index was only covering `memory/` and `MEMORY.md`.

Configuring `extraPaths: ["departments"]` fixed this. Recall@3 jumped from 15% → 20%.

Still terrible, but at least the right files were in scope.

### Document-Side Anchors — +17.5pp

Instead of improving how queries are searched, I tried improving how documents are indexed.

I added a natural-language description to the top of each file—one sentence explaining what questions this document answers:

```markdown
<!-- auto-anchor-v1 -->
> This document answers: What is the current status of the Zn-Mn-Li project? 
> What experiments have been completed? What are the next steps?
<!-- auto-anchor-v1 -->
```

**Result: Recall@3 jumped from 20% → 37.5%. +17.5pp.**

This worked because embedding models perform much better when the text explicitly states its own relevance rather than embedding it implicitly in structured content.

The pattern generalizes: document-side optimization is more durable than query-side optimization, because it persists across all future searches.

---

## The Real Fix: Switching the Embedding Model

### Why the Original Model Was Wrong

The original setup used `nomic-embed-text`—a strong English embedding model. My knowledge base is ~60% Chinese, 40% English.

`nomic-embed-text` was trained primarily on English corpora. On Chinese semantic distinctions, it was essentially producing noise vectors.

### Model Comparison (40 queries, same gold set)

| Model | Recall@3 |
|-------|----------|
| nomic-embed-text (baseline) | 15.0% |
| + extraPaths fix | 20.0% |
| + anchors | 37.5% |
| Gemini text-embedding-001, no anchors | 65.0% |
| Gemini text-embedding-001, with anchors | 67.5% |
| bge-m3, no anchors | 65.0% |
| **bge-m3, with anchors** | **72.5%** |
| **bge-m3, anchors, expanded gold set (121 queries)** | **82.6%** |

Switching from `nomic-embed-text` to `bge-m3` (a multilingual model, 568M parameters) improved recall by **+50pp before any other changes**.

`bge-m3` runs locally. No API costs. The model is 1.3GB. On my RTX 5090, reindexing 120+ files took under 2 minutes.

---

## Automating the Anchor Generation

Manually writing anchor descriptions for 94 files wasn't feasible. So I used a local LLM to do it automatically.

Setup:
- Model: `qwen3:8b` running via Ollama on the RTX 5090
- Task: read each file, generate a natural-language description of what questions it answers
- Output: write back to file in the anchor format

```python
def generate_anchor(content: str) -> str:
    response = ollama.chat(
        model="qwen3:8b",
        messages=[{
            "role": "user",
            "content": f"Read this document and write ONE sentence describing what questions it answers. Start with 'This document answers:'\n\n{content[:3000]}"
        }]
    )
    return response['message']['content']
```

94 files processed overnight. Total cost: $0.

---

## What I Learned

**1. The embedding model is infrastructure, not configuration.**

Choosing the wrong embedding model is like using an English-only dictionary to look up Chinese words. No search parameter fixes this. Audit your model against your actual language distribution before anything else.

**2. Document-side optimization beats query-side optimization.**

Anchors outperformed every parameter I tuned. And they compound: the improvement is permanent and scales with every new query. Writing one good anchor is worth more than 10 parameter experiments.

**3. Heterogeneous knowledge bases have unique failure modes.**

My base had a category of "universal" files (MEMORY.md, AGENTS.md) that appeared in almost every result—not because they were relevant, but because they contained many topics. Adding category-aware filtering fixed this.

**4. Small local models are good enough for annotation tasks.**

`qwen3:8b` generated useful anchors. For structured annotation tasks with clear output format, 8B parameters is sufficient. You don't need GPT-4 for every step in the pipeline.

---

## Current State

The system now runs on `bge-m3` with auto-generated anchors across all files. Real-world recall (on the original 40 queries) sits around **72.5%**, and on the expanded 121-query gold set at **82.6%**.

This is still not perfect. The remaining failures cluster around:
- Queries about very recent events (files not yet indexed)
- Ambiguous queries that could match multiple valid documents
- Cross-lingual queries (Chinese question, English document)

Those are worth a follow-up experiment. But for now, Lulu actually remembers what I told her.

---

*Next post: I've been using bge-m3 locally for a week. Here's a comparison of local vs. API embeddings for personal knowledge bases—latency, cost, and quality tradeoffs.*
