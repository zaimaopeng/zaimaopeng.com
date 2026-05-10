---
title: "The ECONNREFUSED loop that audited my migration"
date: 2026-05-10T01:30:00+10:00
lastmod: 2026-05-10T01:30:00+10:00
description: "I told my AI agent the workstation was 'completely transferred' to my Windows box. It was — at the layer I could see. A single ECONNREFUSED loop forced an audit that surfaced thirteen things still wired for the old setup: orphaned services, unregistered sibling agents, broken skill paths, plaintext keys, a stale model. Migration isn't done when you stop typing. It's done when an audit is green."
tags: ["OpenClaw", "AI", "Migration", "Audit", "Workspace", "Infrastructure"]
categories: ["Tech"]
showTableOfContents: true
slug: "econnrefused-audited-my-migration"
---

# The ECONNREFUSED loop that audited my migration

A few weeks ago I retired my MacBook as a worker for my OpenClaw setup and consolidated everything onto a single Windows workstation. I told the AI: "Mac side is fully gone, you don't need to keep that node config alive." I cleaned what I thought were the relevant pieces. I moved on.

Then last night I sent it a screenshot.

The screenshot was the OpenClaw gateway console, scrolling forever:

```
node host gateway connect failed: connect ECONNREFUSED 100.76.94.64:18789
node host gateway closed (1006)
node host gateway connect failed: connect ECONNREFUSED 100.76.94.64:18789
...
```

I asked the agent to fix the bug.

What I expected: identify the failing process, kill it, move on. Maybe ten minutes.

What I got: six hours, thirteen fixes, and a clear lesson — **the bug wasn't a bug. It was a deferred audit, finally cashing in.**

## Why I thought I was done

When I migrated the workstation, my mental model was simple: "everything important now runs on Windows; the Mac is offline; the workspace files are all here."

That model was true. It was also incomplete.

The migration touched the layer I could see — files, projects, the active gateway, the skills I used daily. Below that lived a second layer of **assumptions** I'd built up over a year of running the system:

- a Windows Startup-folder shortcut auto-launching `node.cmd` (a Mac-era headless worker) at every login
- a `node.json` config file pointing at the *old* Tailscale endpoint
- a skill-mirror watcher daemon running in the background, maintaining stale junctions
- agent directories on disk that never got registered in the canonical `agents.list`
- API keys in plaintext config files, with rotated copies in nine `.bak` snapshots
- a default model frozen one minor version behind the current official recommendation

Each of those was, individually, "kind of fine." The system kept running. Nothing broke loudly enough to announce itself.

Until one of them did.

## The bug that pulled the thread

The ECONNREFUSED loop was caused by exactly one thing: the orphaned `node.cmd` script. It was launching every login (auto-start I'd never noticed), reading a stale config that pointed at this same machine's Tailscale IP on port 18789, and trying to connect.

The gateway *was* on this machine, listening on port 18789. But it was listening on `127.0.0.1`, not on the Tailscale interface. Every connection attempt got refused. The script retried. Forever.

The narrow fix was four commands: stop the process, `openclaw node uninstall`, remove the Startup shortcut, restart. Maybe ten minutes.

But the moment I started looking, the pre-flight `openclaw doctor` check was already screaming about other unrelated issues. *Codex sibling agents on disk but not in `agents.list`. `plugins.allow` empty. `groupAllowFrom` missing. Orphan transcript files. Plaintext API keys.*

Each of those, traced back, came from the same era — the migration window. None of them had been audited.

So I stopped trying to fix the one bug and started pulling the thread.

## The thirteen things that surfaced

| # | Issue | Migration-era origin |
|---|---|---|
| 1 | `node.cmd` looping forever via the Startup folder | Mac worker decommission was scripted, but the Windows Startup shortcut was created as part of an earlier test and never removed |
| 2 | Discord streaming animation off | Carried from a setup where streaming UI didn't matter — but Discord *is* now the primary surface |
| 3 | `plugins.allow` whitelist empty | When the bundled-provider discovery model was tightened, no one ran the migration to set the allowlist |
| 4 | 28+ custom skills silently rejected by symlink-escape | Skill loader hardened against symlink escape; the workspace's mirror layout violated the new boundary |
| 5 | Nine forked skill copies in two locations | Two-stage refactor left old `~/.openclaw/skills/` and new `workspace/capabilities/skills/` both maintained as if canonical |
| 6 | Codex sibling agents unregistered | `agents/codex-ops/` and `agents/codex-worker-harness/` had auth state and sessions but no `agents.list` entry |
| 7 | 245 orphan transcript files | Sessions deleted from the index but not from disk |
| 8 | `cron/jobs.json` reset, but historical run logs lingered | Manual cleanup left `runs/` orphaned |
| 9 | Plaintext `VENICE_API_KEY` and `ANTHROPIC_API_KEY` in main config | Initial wizard wrote them as plaintext; migration to env-var indirection was scoped but never executed |
| 10 | The OAuth token, in plaintext, copied into nine rotating `.bak` snapshots | Auto-backup amplified the leak |
| 11 | `bundledDiscovery` stuck in legacy compatibility mode | Schema upgrade shipped a "compat" intermediate; nothing forced graduation to "allowlist" |
| 12 | Default model frozen at `gpt-5.4` while official guidance pointed at `gpt-5.5` | Model bump skipped because nothing was loudly broken |
| 13 | A dead skill-mirror-watcher Python daemon | Had been running unsupervised for weeks, maintaining stale mirrors that nothing read |

Six were security or correctness issues. Five were performance or UX. Two were cosmetic. None of them was "the bug I reported."

## Pattern: declared done at the boundary I could see

I keep coming back to the same observation: I'd declared the migration "done" at the boundary I could see. Files moved. Mac powered off. Active surface working.

But long-running AI systems have **two boundaries**.

The visible one — the running gateway, the open chat, the skills firing on demand. If those work, you say "done."

The invisible one — every config that *would* matter under different conditions. Auto-start scripts that fire only at login. Watchdog processes that wake when the gateway crashes. Backup files that exist to be read during recovery. Doctor checks that only surface their findings when invoked. Schema validators that warn without blocking.

The invisible boundary is much wider than the visible one. And in a year-old AI workspace, it gets wider every month.

When the visible boundary works, the invisible one is silent. When the visible boundary breaks — *that's* when the invisible one shows up. As error loops. As doctor warnings. As surprise behavior under load.

The bug report was just the moment the invisible boundary stopped being invisible.

## Migration is an audit, not a moment

The mental model I'm carrying out of this:

> A migration is not done at the moment of cutover. It's done when an audit walks the entire visible-and-invisible state and reports green.

Concretely, for an OpenClaw-style local AI system, the post-migration audit checklist looks roughly like this:

- `openclaw doctor` — clean
- `openclaw security audit` — clean
- `agents/` directory inventory matches `agents.list`
- `cron/`, `flows/`, `delivery-queue/` — no orphans
- All plaintext credentials moved to env-var indirection
- All `.bak` / `.clobbered` / staging snapshots reviewed for leaked secrets
- All skill paths actually loading (no silent symlink-escape rejections)
- All scheduled tasks and Startup-folder entries known and intentional
- Default model and plugin allowlist match current official recommendations
- All `nodes/` and `devices/` pairing records match active workers

If any of those are red, the migration is *still in progress* — even if the visible surface is fine.

The work between "things mostly run" and "things actually clean" was where the cost lived. It wasn't enormous, but it was real, and I'd been carrying it as silent technical debt for weeks.

## Anti-pattern: trusting the manifest

I want to call out a specific failure mode I caught myself in.

When I told the agent "Mac is fully gone," I'd updated my `CLAUDE.md` to reflect that. I felt it had been recorded. The agent could read it. The doctrine was current.

But **the doctrine was a manifest, not a state**. It described the world I intended. It didn't audit the world I had.

The dangerous form of this is: a manifest gets updated, then the writer trusts the manifest as if it were a verifier. "I wrote that the Mac is gone, so the Mac is gone." But manifest ≠ system state. Migration is judged on system state, not on doctrine.

The fix isn't to stop writing manifests. They're useful. The fix is: **after a migration, the audit runs against the system, not against the manifest.** And the audit is what closes the migration — not the manifest update.

## What's next

My OpenClaw is now in a state I'd describe as "audited clean" rather than just "running":

- Doctor: zero warnings (one informational note)
- Skills: 41 eligible, single canonical home, no forks
- Agents: all on-disk directories registered, no orphans
- Secrets: zero plaintext in any config or backup
- Model: latest official recommendation, tracking the parity work the platform shipped specifically for it
- Backups: scoped, timestamped, rollback-ready

More importantly, I now have the audit checklist itself. Next time I migrate something — a workstation, a domain, a sub-agent — that checklist runs as part of the migration, not after the next bug surfaces.

The discipline I want to internalize:

> If you didn't run the audit, you didn't finish the migration.

That's not a productivity tip. It's how long-running systems stay clean. Every "I think we're done" without the audit is just a deferred bug report — and the deferral charges interest.

---

*This audit was driven through interactive Claude Code on the Windows workstation, working against the OpenClaw gateway at `127.0.0.1:18789`. Total elapsed: about six hours of guided investigation; total touched: thirteen distinct fixes, roughly ten megabytes of cruft purged, single-source skill consolidation across forty-one active skills.*
