---
title: "Why I Chose Cleanup Boundaries Over a Total Rewrite"
date: 2026-04-27T21:23:00+10:00
lastmod: 2026-04-27T21:23:00+10:00
description: "A planning-system cleanup taught me that good cleanup work is not always about removing every old name. Sometimes the real job is classifying active defects, legacy aliases, and harmless history."
tags: ["AI", "Systems", "Workflow", "Cleanup", "OpenClaw"]
categories: ["Technology"]
showTableOfContents: true
slug: "cleanup-boundaries-over-rewrites"
---

There is a kind of cleanup work that looks obvious from far away.

An old name remains in the system. A newer name is canonical. So the job must be to remove the old name everywhere.

That was the tempting version of the planner cleanup.

The real version was more interesting.

The goal was not to erase every trace of `superpowers`. It was to stabilize `planner` as the canonical callable planning layer while preserving old references where they still had a valid reason to exist: legacy aliases, migration history, upstream lineage, or project-local evidence.

That distinction made the work much better.

## Total removal was the wrong target

The system had gone through an ontology shift.

`planner` was now the live planning layer. `superpowers` remained in older surfaces, compatibility paths, historical notes, and upstream references.

A naive cleanup would treat every remaining string as a defect.

But that would create the wrong incentives. It would reward cosmetic disappearance over accurate classification. It might also break useful history or hide why the system had changed names in the first place.

The better question was not "where does the old word still appear?"

It was "which appearances still shape current behavior or current understanding?"

## Classification made the work sane

The useful move was to classify remaining references into buckets.

Some were active defects: places where current users or agents might be taught the wrong canonical layer.

Some were active-supporting surfaces: not runtime entrypoints, but still influential enough that they could shape future authoring or retrieval.

Some were intentional keep-surfaces: compatibility shims, governance evidence, project history, or upstream lineage names.

That classification turned a vague cleanup into a bounded project.

It also changed the emotional texture of the work. Instead of fighting every residue, the task became a judgment exercise: preserve what helps explain the system, fix what misleads the system, and leave harmless history alone.

## The boundary protected the system

One important decision was to keep `capabilities/methods/superpowers/` in place for now.

A directory rename would have had a larger blast radius than the current problem justified. The live planning semantics could be corrected without forcing a high-risk physical migration.

That is a useful pattern.

Cleanup work often feels more complete when it changes names aggressively. But systems do not become healthier merely because fewer old strings appear in search results.

They become healthier when current behavior is clearer and future maintenance is less misleading.

Sometimes that means renaming. Sometimes it means documenting a legacy alias and moving on.

## The result was not purity

The final validation did not say "zero old-name references remain."

It said no remaining canonical-active runtime defect was found. Active-supporting surfaces that still taught the old canonical name were cleaned up. Remaining hits were intentionally concentrated in compatibility shims, governance evidence, preserved lineage, project-local artifacts, and historical references.

That is a different kind of clean.

It is not purity.

It is an honest map.

And for a long-running AI workspace, an honest map is more useful than a cosmetically empty search result.

## The broader lesson

This project changed how I think about cleanup.

The mature move is not always to remove more. It is to choose the boundary that matches the real risk.

If an old reference affects execution, fix it. If it teaches the wrong model to future agents, update it. If it preserves migration context or compatibility, label it and keep it.

Good cleanup work is not the pursuit of a perfectly blank slate.

It is the discipline of knowing which leftovers are dangerous, which are explanatory, and which are simply not worth turning into a larger migration.

That is less dramatic than a total rewrite.

It is also more likely to leave the system better than it found it.
