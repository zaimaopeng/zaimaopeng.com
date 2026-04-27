---
title: "Why Upstream Blog Automation Needed Its Own Workflow"
date: 2026-04-27T21:20:00+10:00
lastmod: 2026-04-27T21:20:00+10:00
description: "A real blog automation run taught me that candidate discovery, article drafting, and publishing are different jobs. Bundling them together makes automation faster, but also easier to trust too soon."
tags: ["AI", "Automation", "Blogging", "Workflow", "OpenClaw"]
categories: ["Technology"]
showTableOfContents: true
slug: "upstream-blog-automation-workflow"
---

I originally wanted the blog workflow to feel simple: look at recent work, find something worth writing, turn it into a post, and publish it.

That sounds like one workflow.

In practice, it became three different workflows wearing the same coat.

The first workflow is discovery: scanning project logs, closeouts, daily notes, and handoff files to find real story signals. The second is authoring: turning one signal into a coherent article with a hook, claim, evidence, and shape. The third is publishing: syncing the production bundle into the Hugo site, checking output, pushing the commit, and verifying the live page.

The lesson from building `blog-automation` was that these jobs should stay separate.

## The first version worked, then showed its boundary

The first useful version of the automation did something real. It generated candidate article packages from recent project work. Those candidates were not just theoretical. Several went through the downstream blog workflow and became live posts.

That was the strongest validation: the runner returned `ok`, the queue was readable, and the candidates were good enough to become production writing.

But the same validation also exposed the danger.

Once a system can generate a plausible queue, it is tempting to let it keep moving. A candidate can look like a draft. A draft can look like a post. A pushed commit can look like publication. A green script can feel like truth.

Those are different states.

The automation needed to respect them.

## Discovery is not drafting

Upstream automation is useful because it can notice patterns that are easy to miss in the middle of work.

A project closeout might contain a stronger article than the project plan. A daily log might reveal the real failure mode. A handoff note might preserve the exact sentence that explains why the work mattered.

That is a discovery problem.

It asks questions like:

- Is there a real change in judgment here?
- Is this just status chatter, or did something break and teach a rule?
- Is the angle public enough to write about?
- Has this already been published under another slug?

Those questions are not the same as article drafting.

Drafting needs rhythm, compression, section order, bilingual judgment, and a clear reader path. Discovery only needs to produce a strong enough package for a human or downstream workflow to decide what deserves that attention.

When those two layers blur, the system starts producing writing-shaped objects before it has done selection well.

## Publishing is even more different

Publishing has a stricter truth contract.

A post is not published because a draft exists. It is not published because the repo was synced. It is not published because Git accepted a commit. It is published only when the local output and the live site both verify the page.

That is why `blog-automation` should not own publication.

Its job is to keep the candidate queue fresh, honest, and non-duplicative. `blog-publishing` owns the article, the production bundle, the sensitivity checks, the Hugo validation, the commit, the push, and the live verification.

That split may look slower, but it makes the system easier to trust.

## The real hardening came from drift

The most useful bug surfaced after the first few successful runs: already published topics came back into the queue.

The generator had correctly detected overlap and wrote a warning into the candidate notes. But it did not exclude the candidate. So the queue could say "high overlap risk" and still list the article as active.

That was a small implementation bug with a large workflow lesson.

Warnings are not state transitions.

If a candidate has the same slug as a published post, the system should not merely describe the problem. It should stop that candidate from entering the active queue.

That fix made the automation more honest. It also clarified the role of the queue: an active queue should contain work that is actually eligible for review, not known-invalid items with a caution label attached.

## What I want from this layer

The goal is not full auto-publishing.

The goal is a better upstream memory for writing: a queue that keeps scanning real work, filters out already published ideas, packages strong candidates, and hands them forward only when they deserve more effort.

That is the layer I was missing.

Without it, blog ideas stay scattered across project logs and daily notes. With too much automation, weak or duplicated topics move too quickly toward publication. The useful middle is a candidate workflow that is selective, explicit, and reviewable.

That is why upstream blog automation needed its own workflow.

It is not more publishing glue.

It is the part of the system that decides what is worth publishing in the first place.
