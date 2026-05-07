---
title: "Strip the personalization first: how I made my OpenClaw governance portable"
date: 2026-05-07T19:30:00+10:00
lastmod: 2026-05-07T19:30:00+10:00
description: "After running my OpenClaw workspace for the better part of a year, it became a machine fitted to me — domains, skills, charter, routing tables, all carrying my personal residue. Trying to share it with another OpenClaw user, my first instinct (just copy it) immediately broke. This post is about why governance can't be moved but mechanism can — and the seven plugins I extracted to prove it."
tags: ["OpenClaw", "AI", "Agent", "Workspace", "Plugin", "Governance", "Infrastructure"]
categories: ["Tech"]
showTableOfContents: true
slug: "portable-openclaw-by-extracting-personalization"
---

# Strip the personalization first: how I made my OpenClaw governance portable

My OpenClaw workspace has been running for about nine months.

It has five domains, dozens of skills, a charter, several routing tables, and a pile of playbooks, templates, and audit scripts I worked out for myself by trial and error. It became a machine fitted to me: it knows when to default to Chinese vs English, knows which files my sub-agents must not read, knows when I want deep-work and when I just want to scan.

Recently I got curious about a question:

> Could I share this with another OpenClaw user?

The instinctive answer is: sure, just copy it.

But the moment I tried, the instinct turned out to be wrong.

What I can hand to someone else is **not** my whole workspace. It's what's left of my workspace **after the personalization is stripped out**. The gap between those two things turned out to be much bigger than I'd expected.

## Why "just copy the workspace" doesn't work

The most direct approach is obvious: tarball `~/.openclaw/workspace/` and hand it over.

Open it, and the mess is immediate:

- `domains/career/` is full of my research direction, my collaborators, my lab protocols. Useless to anyone else.
- `CHARTER.md` opens with my studio name. Defines `Lulu` as my cloud mind.
- `MEMORY.md`'s routing table assumes my five domains. Rename them and the whole thing breaks.
- `USER_PRIVATE.md` is private. Must never leak.
- Every skill's trigger phrasing assumes my working language habits.
- `runtime/` is full of my own little apps and PowerShell scripts. Nobody else's concern.

Force-feed all of this to someone else and what they'd open is **a copy of someone's life**, not a governance system. It's noise.

But here's what's interesting: that pile **does** contain the genuinely valuable, reusable parts.

They're just tangled with my personal content.

## What's actually movable isn't files — it's mechanism

The tangle starts to come apart once I separate "workspace" from "governance system."

My workspace is the **content layer**. Specific projects, specific research, specific clients, specific preferences.

My governance is the **mechanism layer**. Things like:

- "Identify object type first, then read the smallest relevant surface" — the three-step routing rule
- "Exactly 1 primary, 0–1 planner, 0–1 mode, ≤2 support" — the skill-loading guardrails
- "domains/, projects/, capabilities/, runtime/, archive/, memory/, inbox/" — the seven-layer physical layout
- "🔴 / 📋" — the two-flag dashboard convention
- "Domains have six governance states" — the lifecycle vocabulary
- "Every active project has a `PROJECT_LOG.md`, updated before closeout" — an iron rule
- "Add `<!-- auto-anchor-v1 -->` blocks with hypothetical queries to every governance file" — the retrieval-augmentation convention

The mechanism layer doesn't depend on my specific content. It's **orthogonal** to my life.

So what's actually transferable to someone else is the mechanism — and what I'd been conflating was binding the mechanism to my own instantiation of it.

The extraction project is, fundamentally, the work of pulling those two layers apart.

## The seven extracted artifacts

I spent an afternoon doing exactly this: walking the governance layer and pulling each mechanism out of my personal OpenClaw, into **seven independent OpenClaw plugins**.

| Plugin | Mechanism extracted |
|---|---|
| `openclaw-skill-taxonomy` | 7-kind classification, selection guardrails, `skill-router` planner, `skill-author` executor |
| `openclaw-workspace-ontology` | 7-layer object model (domains/projects/capabilities/runtime/archive + memory/inbox), object-type-first routing, promotion rule |
| `openclaw-domain-ops` | What a domain is, six governance states, open/split/merge/deprecate/archive ops, domain README templates |
| `openclaw-project-ops` | Project lifecycle + `/endproject` shim + project templates + closeout/health-audit playbooks |
| `openclaw-memory-anchors` | `<!-- auto-anchor-v1 -->` protocol, hypothetical-queries authoring guide, anchor lint script, episodic-memory health audit |
| `openclaw-charter-pattern` | 6-clause charter structure, 🔴/📋 flag conventions, daily/weekly briefing skeletons |
| `openclaw-inbox-ops` | Inbox CONTRACT (queue / mail / outbox three lanes) + intake/promotion playbooks |

Each plugin ships its own `openclaw.plugin.json` install contract, its own README, its own templates and scripts. An installer can pick `#1 + #2` (the minimum useful stack) or all seven.

But **none of these plugins contain anything of mine**.

The `CHARTER.md.tmpl`'s studio name is `<<WORKSPACE_NAME>>`. The `MEMORY.md.tmpl`'s domains are `<<domain-1>>`. My name in the permission table is `<<PRINCIPAL_NAME>>`. `USER.md`, `IDENTITY.md`, `SOUL.md` — those simply **didn't enter** the extraction at all.

The whole process was driven by one repeated motion: identify a mechanism, find every "instantiation footprint" of it in my personal workspace, and replace each footprint with a placeholder.

If I couldn't, that meant the thing wasn't mechanism. It was personalization. And it didn't belong in the extraction.

## Personalization is friction

I started this thinking it would be 80% engineering tidying. Turned out it was 80% **discrimination**.

For every governance file, I kept asking:

- Is this clause mechanism, or personalization?
- Is this rule generalizable, or only useful given my work patterns?
- Is this skill name an abstract role, or my own well-worn shorthand?
- Is this iron rule from universal experience, or from a specific scar I happen to carry?

Anything that leaned "personalization" got either deleted, marked as `<<placeholder>>` for the installer to fill, or moved into a "case study" section labeled "this is what Michael does — adapt, don't copy."

After enough of this, I felt a very specific fact in my hands:

> Personalization is friction.

It's not a soft "additional consideration." If a governance system retains references to the original author's life, hints about the original author's workflow, or dependencies on the original author's specific toolchain, its reusability **stalls**. The amount of stall is proportional to the residue.

A lot of "I'm sharing my prompt / workflow with everyone" attempts in the AI space stall for exactly this reason. The shared artifact looks generic, but the moment someone else installs it they hit a wall of "what does this name mean / why is this rule written this way / I don't have this folder." That dull pain is the personalization residue exposed.

The first job in extraction isn't writing new code. It's **stripping**.

## An anti-pattern: forking your whole workspace

Throughout this work I kept benchmarking against an anti-pattern.

That anti-pattern: **package your entire workspace as a "starter template" for others**.

It sounds generous — give them every battle-tested structure and let them fork from it.

But it actually shoves the discrimination work onto every installer. They open it and face a wall of "looks professional but no idea what to change, what's mechanism, what's Michael's life," and then trial-and-error their way out. By the time they figure it out, they've essentially redone the extraction work I already did — without the manifest, without the SKILL.md frontmatter, without the constraints.

The anti-pattern is seductive because it's **easiest for the author**.

It's **most expensive for every installer**, each of whom has to redo the stripping.

So the correct path is the opposite: the author does the stripping once, packages the result as manifest-bound plugins, and what every installer receives is mechanism with the personal residue already gone.

The trade-off — "author works twice as hard so every installer works less" — is the trade-off anyone trying to build "portable AI governance" has to make.

## This is a new kind of infrastructure for the AI agent era

Stepping back for a moment.

OpenClaw — a long-running, multi-tool, multi-agent local AI OS — is becoming a new kind of infrastructure. It's not centralized like a cloud service. It's not static like traditional software. It's **a slightly different tree growing in every user's house**.

Every OpenClaw is personalized. That's fine.

But if "personalization" and "mechanism" grow tangled together, the tree only works for its grower. Nothing crosses between users. Every newcomer plants from seed. Every scar everyone earns gets re-earned by everyone else. The whole ecosystem can't scale.

Extraction is the counter-force.

It assumes: **mechanism is shareable; personalization is what hides it**.

If you keep separating them out — via manifests, via placeholders, via plugin boundaries — then OpenClaw users can share at the mechanism layer while diverging at the content layer. A bit like Unix: every home directory is unique, but `/usr/` and `/etc/` ship a shared mechanism set.

We didn't really do this in the prompt-engineering era because the AI was stateless. Every conversation rebooted.

But OpenClaw is **stateful, memory-bearing, runs in your house, and evolves over time**. It has `domains/`, `projects/`, `memory/`, `inbox/`. What it needs isn't better prompts. It's a **reusable governance layer**.

And reusable governance requires that the personalization be stripped out first.

## What's next

My own OpenClaw won't change because of this. It looks the same as before.

But I now have a publishing surface.

Next time someone wants to build their own long-running OpenClaw, they don't have to think from scratch about "how should I organize domains / will my skills fight each other / where should project lessons get promoted." They can install these seven plugins — or at minimum the two-plugin minimum stack — and start using them, growing their own content on a stable mechanism skeleton.

And as for me — having pulled the mechanism layer out of my workspace, I now see "my part" of it more clearly than I ever did.

That turned out to be more valuable than I expected.

The act of stripping is, in itself, an honest audit of one's own working system.

---

*The seven extracted plugins, plus INVENTORY and DESIGN docs, now live at `~/.openclaw/plugin-extraction-2026-05-07/`. If you're running OpenClaw and want to try them, start with the minimum stack: `openclaw-skill-taxonomy` + `openclaw-workspace-ontology`.*
