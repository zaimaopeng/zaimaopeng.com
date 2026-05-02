---
title: "I Used WOOP to Make My AI Agents Stop Drifting"
date: 2026-05-02T10:27:21+10:00
lastmod: 2026-05-02T10:27:21+10:00
description: "Turning WOOP into an agent workflow contract taught me that the real win is not another planning template. It is making drift visible, recoverable, measurable, and falsifiable."
tags: ["AI", "Agents", "WOOP", "Workflow", "OpenClaw"]
categories: ["Technology"]
showTableOfContents: true
slug: "woop-ai-agents-stop-drifting"
---

While updating my `Deep Work` method, I noticed a pattern that felt uncomfortably familiar:

many AI agents do understand the user's wish.

They can restate the goal. They can produce a plausible plan. They can sound aligned.

And then they drift during execution.

Drift has many shapes. A planner gives five equal options but never recommends one. A research agent keeps gathering context after there is enough evidence to synthesize. A coding agent fixes a script but does not rerun the validation cleanly. A project-ops agent keeps a project falsely open because closeout feels ambiguous. A deep-work agent starts reporting what it will do next instead of continuing to produce artifacts.

That resembles human drifting: the wish exists, but the trigger for action has been handed over to defaults, habits, local reactions, and the surrounding environment.

So I tried a small experiment:

I turned WOOP from a human self-regulation technique into an agent workflow contract.

WOOP, popularized by Gabriele Oettingen, stands for Wish, Outcome, Obstacle, Plan. It asks you to name the desired future, contrast it with the real obstacle, and bind the obstacle to an If-Then response.[^woop]

For a person, it looks roughly like this:

```text
I want X.
I want the result Y.
I am most likely to get blocked by Z.
If Z appears, I will do A.
```

For an agent, I translated it into:

```text
What deliverable is the user really asking for?
What evidence would show that the outcome has been reached?
What internal agent pattern is most likely to cause drift?
If that drift signal appears, what recovery action must run first?
```

The third line is the important one.

The Obstacle cannot be "the environment is complicated." For an agent, the useful obstacle is agent-side: its own failure mode.

For example:

- The planner's obstacle is not "many choices"; it is "using option lists as a substitute for recommendation."
- The research agent's obstacle is not "insufficient information"; it is "continuing collection after evidence is already sufficient."
- The coding agent's obstacle is not "the codebase is complicated"; it is "soft-closing after failed validation."
- The deep-work agent's obstacle is not "the task is long"; it is "repeating plans and forgetting the deliverable."

At that point WOOP stops being a motivational acronym and becomes a workflow contract.

## Where I Integrated It

The first target was `Deep Work`.

I added an internal WOOP loop that runs silently at kickoff, stage transitions, validation failures, and final closeout. The agent checks Wish, Outcome, Obstacle, and If-Then Plan before it reports success.

But WOOP clearly did not belong only to Deep Work.

It was more useful as a shared alignment method, so I integrated it into four other drift-prone skills:

- `planner`: run WOOP before route selection to reduce option dumping and over-planning.
- `coding-agent`: run WOOP before ambiguous or multi-file fixes to control patch scope and validation behavior.
- `auto-research`: run WOOP at intake and stop/continue decisions to prevent endless collection.
- `project-ops`: run WOOP when opening, steering, or closing projects to avoid false-open states and over-promoting unfinished material.

The important change was not that the skills got longer.

The important change was that each skill gained an executable recovery sentence:

```text
If I start doing drift pattern X, then I must first do recovery action Y.
```

That is the logic of implementation intentions, or If-Then planning. The point is not merely to think harder about a goal; it is to bind a situational cue to a concrete response in advance.[^gollwitzer]

For an agent workflow, this acts like a small interrupt handler.

## The Scores

I ran three layers of evaluation.

The first was a manual scorecard asking whether the skill contracts became better at constraining drift.

Result:

```text
36.8 -> 47.2 / 50
```

The second was a deterministic contract-coverage harness. Instead of asking whether the change "felt better," it compared the baseline instruction surface from git `HEAD` against the current worktree and checked whether executable drift guards were actually present.

Result:

```text
48.0 -> 60.0 / 60
relative improvement: 25.0%
```

The third was a current-thread live replay pack with five behavior cases: Planner, Coding Agent, Auto Research, Project Ops, and Deep Work.

That single-arm replay scored:

```text
45 / 50 = 90%
```

I am using the phrase "single-arm" deliberately.

It shows that under the current WOOP condition, the workflow can produce artifacts, validation evidence, bounded research, and honest limitation statements during a real task.

It is not yet blinded A/B proof.

That distinction matters.

## Did I Design the Score to Make Myself Look Good?

This is the most important part of the story.

If someone changes an architecture, designs a score afterward, and then announces that the new architecture scored better, the obvious critique is:

```text
Did you just design the metric to reward your own intervention?
```

That critique is fair.

It belongs inside the method, not outside it.

So I added a validity audit. Its opening challenge is essentially:

```text
Did the evaluator design a scoring system after the integration in order to make the new architecture look good?
```

My defense is not "trust me."

The real defense is to make the result capable of losing.

The current safeguards are:

- The baseline comes from git `HEAD`, not a hand-written weak strawman.
- The deterministic harness and raw outputs are stored and rerunnable.
- The live replay rubric does not reward saying the words Wish, Outcome, Obstacle, or Plan. It rewards artifacts, validation, scope control, failure recovery, and honesty about limits.
- There are negative controls: for example, safety-gate integrity should not magically jump just because WOOP was added.
- There are falsification criteria: if WOOP increases user-drag, increases plan-only loops, lets research continue forever, or lets coding soft-close after failed validation, the behavioral claim has to be narrowed or rejected.
- The strongest current claim is contract-level improvement, not production telemetry.

This is related to Goodhart's Law: when a proxy metric becomes the target, the system can learn to game the proxy.[^goodhart]

So a good evaluation should not merely let you win.

It should let you lose.

The next experiment is blinded A/B replay. Run the same prompts against baseline and WOOP conditions, randomize the outputs as Run A and Run B, score them without knowing which is which, and only then unblind.

That borrows from the spirit of registered reports and blinded interpretation: freeze the method before looking at the result, and lock the interpretation before revealing the condition.[^registered][^blind]

Only then would I be more comfortable making a behavioral-performance claim.

## The Actual Lesson

This experiment made me more convinced of one thing:

many agent problems are not failures of intelligence. They are failures of execution precision.

An agent can understand the wish and still drift during execution.

What it needs is not a longer plan. It needs a recovery action that fires when drift appears.

That is where WOOP helped.

It compiled an abstract wish into an executable If-Then:

```text
If I start moving away from the Outcome,
then I handle the Obstacle first,
and only then continue the Plan.
```

For a person, that is a way to reclaim the trigger for action.

For an agent, it is a way to turn "knowing the goal" into "returning to the goal during execution."

I do not want to frame this as "WOOP has proven an X% production improvement across all agents."

That would be too early.

The more honest and more useful claim is:

WOOP made my agent workflow contracts more executable. It turned drift from a vague feeling into something nameable, triggerable, measurable, and falsifiable.

That is already important.

Because the most dangerous agent state is often not failure.

It is looking busy while quietly drifting away from the user's real outcome.

[^woop]: WOOP is commonly described as Wish, Outcome, Obstacle, Plan; see the public WOOP resource at [woopmylife.org](https://woopmylife.org/).
[^gollwitzer]: For implementation intentions and If-Then planning, see the University of Vienna publication page on [mindset theory of action phases and if-then planning](https://ucrisportal.univie.ac.at/en/publications/mindset-theory-of-action-phases-and-if-then-planning/).
[^goodhart]: David Manheim and Scott Garrabrant, ["Categorizing Variants of Goodhart's Law"](https://arxiv.org/abs/1803.04585).
[^registered]: Nature's [Registered Reports guidelines](https://www.nature.com/nature/for-authors/registered-reports) describe pre-registering methods and analyses before the research is conducted.
[^blind]: For blinded interpretation as a bias-control method, see ["Blinded interpretation of study results can feasibly and effectively diminish interpretation bias"](https://www.sciencedirect.com/science/article/pii/S0895435613004861).
