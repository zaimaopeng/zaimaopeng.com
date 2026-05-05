---
title: "Why I Pulled Lulu Out of OpenClaw and Turned Her Into a Plugin"
date: 2026-05-05T10:30:00+10:00
lastmod: 2026-05-05T10:30:00+10:00
description: "A recent Discord failure made it clear that Lulu could not remain a pile of personalized rules scattered across OpenClaw config. Every OpenClaw upgrade kept triggering config and plugin compatibility work. Extracting Lulu into a local lulu-profile plugin keeps the official core clean and turns Lulu into something maintainable, upgradeable, and eventually shareable."
tags: ["AI", "OpenClaw", "Lulu", "Agent", "Plugin", "Workflow"]
categories: ["Technology"]
showTableOfContents: true
slug: "lulu-profile-plugin-extraction"
---

<!-- auto-anchor-v1 -->
> **Hypothetical queries:** Why extract Lulu from OpenClaw into a plugin | OpenClaw upgrade compatibility issues | AI agent profile plugin design | Discord failure broke Codex OpenClaw workflow | maintainable AI agent personalization layer | lulu-profile plugin before_prompt_build before_model_resolve | keeping OpenClaw core unpatched | AI work system upgrade pressure
<!-- auto-anchor-v1 -->

This time, I pulled Lulu out of OpenClaw.

More precisely, I moved the parts of runtime behavior that really belong to Lulu into a local OpenClaw plugin: `lulu-profile`.

That sounds like an ordinary engineering cleanup. Gather scattered logic. Clarify ownership. Keep the official package clean. All true. But if I only describe it that way, the story becomes too neat.

The real trigger was less elegant.

The real trigger was that Lulu's Discord path broke again.

Not completely. OpenClaw was not dead. Some entry points still worked. Some configuration still looked right. The plugin registry still showed the expected pieces. But Lulu was not replying correctly on Discord, or she was replying through the wrong path. The Codex and OpenClaw loop I had started to rely on suddenly lost one of its main operating surfaces.

This was not the first time.

Every OpenClaw upgrade had been exposing some configuration or plugin compatibility problem. Not usually one dramatic explosion, but a chain of small misalignments: a plugin install path changed, a config field changed meaning, an external plugin had to become the official version, a thread binding shape was stale, or a model route that used to resolve stopped being reliable after the upgrade. Then Codex and I would spend a long session pulling the system apart: did OpenClaw itself change, did Lulu's local config pollute the core path, or did Discord, model routing, prompt override, cron, and update guards step on each other again?

That was the part I was tired of.

The issue was not that upgrades have bugs. Software upgrades always have compatibility work.

The deeper problem was that Lulu was not a cleanly bounded thing.

## Lulu Was Too Scattered

Lulu did not begin as a plugin.

She grew out of months of using OpenClaw heavily: a system prompt, a Discord-safe mode, a model route, local tool restrictions, workflow discipline, lessons from incidents, and a set of files around memory, projects, skills, and daily operation.

Together, those things became "Lulu."

But they did not all live in one boundary.

Some of her behavior was in `openclaw.json`. Some was in a global prompt override. Some was in Discord account config. Some was in workspace skills and governance files. Some was in cron. Some was in the update guard. Some of it was even worse: knowledge that existed only because I remembered that we had fixed something in a previous upgrade and should be careful this time.

That was acceptable while the system was small.

At the beginning, the main problem was getting her to work at all. Could she reply in Discord? Could she read the workspace? Could she cooperate with Codex on execution and review? Could she reconnect projects, writing, and system state? Could she recover her identity and operating rules from files?

Early systems do not need to be beautiful. They need to become useful.

But Lulu gradually became a long-running work layer.

She was no longer just a chat window. She became the agent surface inside OpenClaw that helped with project flow, article production, workflow routing, incident review, system upgrades, and state checks. When she broke, it was not just that "a bot could not chat." A key part of my local AI work system went numb.

At that point, scattered personalization became debt.

Especially during upgrades.

## Every Upgrade Felt Like System Archaeology

Before this extraction, I had started to see the same pattern over and over.

I wanted to keep OpenClaw close to the official version. If the official package fixed bugs, changed plugin behavior, or improved the runtime, I wanted those changes in my system.

But before every upgrade, I had to ask a tiring list of questions.

Which parts of this config belong to official OpenClaw?

Which parts are Lulu's personalization?

Which parts were added for Discord?

Which parts are safety lines from previous incidents?

Which parts can be removed, which ones must stay, and which local patches have already been absorbed by upstream?

When those questions do not have clear answers, an upgrade stops being an upgrade. It becomes archaeology with live wires.

The worst part is that the more useful Lulu's personalization became, the easier it was to mistake it for OpenClaw core behavior. A global `systemPromptOverride` may look like just a config field, but in practice it carried Lulu's identity, safety mode, Discord reply behavior, and operating boundaries. A model route may look like just a provider and model choice, but it determined whether a Discord turn reached the correct model. Tool restrictions may look like a defensive detail, but they prevented Lulu from using local file, shell, or patch tools during an ordinary Discord writing request.

All of that mattered.

But it did not belong inside the OpenClaw core.

As long as it lived there, I could not say: "Upgrade official OpenClaw, keep loading the Lulu profile."

Instead, every upgrade became: back up the profile, compare config, patch compatibility issues, refresh the plugin registry, inspect Discord, inspect the gateway, run the guard, and then decide which failure belonged to OpenClaw and which one belonged to Lulu's local layer.

That is not sustainable maintenance.

It is a long-running agent identity tied directly to a moving host configuration.

## The Discord Failure Made It Impossible To Ignore

The thing that finally pushed this work over the line was Discord.

I have been relying more and more on a dual-agent loop between Codex and OpenClaw. OpenClaw, through Lulu, holds long-term context and operating entry points. Codex enters bounded workspaces, reads files, patches code, runs checks, and returns verifiable results.

The value of that loop is not that two AIs are working at once. It is that they own different responsibilities.

Lulu owns continuity. Codex owns local execution and external review. I keep the final judgment.

But that loop depends on a very practical condition: Lulu's entry point has to be stable.

When Lulu stops replying correctly on Discord, the problem is not just "the Discord bot is broken." My operating surface breaks. Work can still be done, but the rhythm is damaged. OpenClaw cannot reliably receive intent, Codex cannot smoothly inherit context, and system state cannot move cleanly from one agent back to the other.

When we investigated this time, the problem again had the familiar shape: config, plugins, model routes, Discord delivery, output cleanup, and tool boundaries.

That is when I stopped wanting to repair the current state one more time.

I wanted to repair the structure.

If Lulu's Discord-safe mode, prompt injection, model route, tool blocking, and outgoing message cleanup all belong to the Lulu profile, then they should be owned by a Lulu profile plugin. They should not be scattered across global OpenClaw configuration.

OpenClaw should remain the official host.

Lulu should become a loadable, inspectable, upgradeable, removable profile.

## The Goal Was To Create a Boundary

So Codex and I chose a simple order: extract first, upgrade later.

Not: upgrade OpenClaw first, then rescue the system from whatever compatibility problems show up.

Instead: admit that Lulu is a local personalization layer, and give that layer a boundary.

The result was a local OpenClaw plugin named `lulu-profile`.

Its responsibilities are narrow.

Through `before_model_resolve`, it applies Lulu's model route for Discord turns and sends them to the current production model.

Through `before_prompt_build`, it injects Lulu's safe-mode prompt without relying on `agents.defaults.systemPromptOverride`.

Through `before_tool_call`, it blocks mutating local tools in Discord chat contexts. If someone asks Lulu to polish a sentence or rewrite a message, she should not suddenly edit workspace files or run shell commands.

Through `message_sending`, it cleans outgoing Discord content so internal artifacts like `NO_REPLY`, `/no_think`, or thinking blocks do not leak into the visible chat surface.

It also exposes a `lulu-profile` status command so I can inspect whether the profile is enabled, which agents and channels it targets, which model route it uses, and which safety options are active.

What it does not do is just as important.

It does not own the Discord token. It does not register a model provider. It does not start a background service. It does not write during normal turns. It does not patch `node_modules/openclaw`.

That mattered to me because the goal was not to hide complexity. The goal was to put complexity in the right place.

Lulu's behavior belongs to Lulu's profile.

OpenClaw's official core belongs to OpenClaw.

The connection between them should be small and explicit: load the plugin, enable the plugin, and place the necessary prompt, model, and channel targeting under `plugins.entries.lulu-profile.config`.

## The Upgrade Became Verifiable

Only after that extraction did we continue with the official upgrade.

Moving to OpenClaw `2026.5.2` still had compatibility work. Discord needed the official external plugin. Old config fields needed cleanup. The plugin registry had to be refreshed. The gateway had to be restarted and checked. The guard had to be updated to express the new invariants.

But the shape of the problem changed.

I no longer had to guess where Lulu lived.

She lived in `lulu-profile`.

The official package could remain untouched. The profile patch could stay small. The guard could check concrete invariants: is `lulu-profile` enabled, is the plugin path present, has the global prompt override been removed, does Discord use the right model route, and are output cleanup and tool blocking still active?

The final state was much clearer:

OpenClaw core was upgraded to `2026.5.2`.

Discord used the official external plugin.

Lulu-specific behavior moved into local `lulu-profile`.

The global prompt override was removed.

Old cron jobs were cleared to make room for a cleaner automation rewrite.

The final deep guard reported `75 pass, 0 warn, 0 fail`.

The number itself is not the main point.

The point is that it now proves something meaningful: Lulu is no longer held together by scattered local repairs. She is an inspectable unit.

## Pluginizing Lulu Is Not Removing Her Personality

There is an easy misunderstanding here.

Turning Lulu into a plugin does not make her colder, less personal, or less real as an assistant.

It does the opposite.

A personality and a set of operating habits are more durable when they have a clear boundary.

If Lulu is only a prompt buried in global config, she is fragile. One upgrade, one config migration, or one plugin compatibility change can deform her behavior, and it becomes hard to identify what actually changed.

If Lulu is a profile plugin, she has a nameable, installable, inspectable, backupable, movable shape.

That matters for my system, and it also matters for future open source use.

Not everyone needs my entire OpenClaw workspace. Nobody should copy all of my local paths, incidents, project history, and personal operating rules. But other people may benefit from the pattern of a long-running AI assistant profile: how to own prompt injection, how to prevent local tool misuse in Discord chat, how to route models, how to clean internal directives from outgoing messages, and how to keep personalization separate from the official runtime.

If those pieces stay scattered across my `openclaw.json` and workspace, they remain private infrastructure.

If they become a plugin, they can become something other people can understand and reuse.

That is one of the most interesting side effects of the extraction.

Lulu became easier to maintain.

She also became easier to share.

## The Larger Lesson

Looking back, this work belongs to the same line of lessons as the previous Lulu incidents.

The self-termination incident taught me that an agent must understand not only how to solve a problem, but also where its own runtime boundary is.

The Watchdog episode taught me that "configured" is not the same thing as "reliable."

The state-integrity work taught me that restarts, recovery, and health checks need file-backed evidence and verification. The system cannot simply believe its own story.

This extraction taught me something at a different layer:

A long-running AI agent should not just become an ever-thicker layer of local configuration inside its host system.

If it starts doing long-running work, it eventually needs its own boundary.

Not for elegance.

So upgrades do not require rediscovering the system every time.

So failures can be localized: OpenClaw core, Discord plugin, model route, or Lulu profile.

So the official runtime can keep following official releases while the local personality layer evolves independently.

And so one day, I can share the useful parts without handing people a whole private workspace full of personal paths, historical incidents, and one-off patches.

So this was not just a refactor.

It was the moment Lulu moved from being a local habit grown inside OpenClaw to being an independent profile running on top of OpenClaw.

That sounds like an engineering boundary.

For a long-running AI assistant, the boundary is part of the survival model.
