---
layout: post
title: "The 6 Phases of a Programmer's Adaptation to AI"
date: 2026-06-07
description: "How the relationship between a developer and AI agents evolves — from manual copy-pasting to automated validation."
---

## The 6 Phases of a Programmer's Adaptation to AI

## Phase 1: Curiosity and Manual Copy-Pasting

It all starts with curiosity. You download a local LLM, open a chat, and start asking it things. You copy answers by hand. Sometimes they work, sometimes they don't. But curiosity is enough to keep you interested.

The flow is simple: you think, the model responds, you copy. There's no automation, no integration. It's like having a pair programming partner who talks slowly and hallucinates sometimes. But it's enough to experiment.

## Phase 2: Frustration

Then comes the reality check. Small local models promise a lot but deliver very little. They generate code that looks fine but has subtle bugs. They break imports. They forget contexts. Productivity drops below what you'd have without AI.

This is the "AI slop" phase: generated code nobody understands, nobody trusts, and nobody wants to maintain. You start doubting. Is this worth it? Or are we just playing with expensive toys?

## Phase 3: The Qualitative Leap

Then comes the moment when something changes. A new model, a leap in reasoning quality, and suddenly things start working. Not everything, but enough for the friction to disappear.

It's like when you went from writing code by memorizing syntax to understanding what you wanted to express. The tool disappears and what remains is the work.

## Phase 4: The Virtual Team Illusion

Once you trust agents, the natural instinct is to replicate what you already know: teams with roles. You create a "PM" agent that plans, a "dev" agent that codes, a "QA" agent that tests. It seems natural. It makes sense.

But soon you discover it doesn't work. Agents aren't people. They don't need functional role divisions — they need *cognitive depth* divisions. It's not that one "plans" and another "codes". It's that some tasks require deep reasoning while others require mechanical execution.

The most common mistake is treating agents as mini-people. They aren't. They're tools with different levels of power. And mixing them for organizational convenience rather than cognitive necessity is the perfect recipe for chaos.

## Phase 5: The "Man-in-the-Middle" Trap

When the virtual team illusion collapses, many programmers fall into the opposite trap: reviewing everything. Every diff, every line, every decision. At first it seems responsible. You're being careful.

But soon you discover the paradox: while the agent works, you do nothing. When you work (reviewing), the agent is idle. You're creating a human bottleneck where there shouldn't be one.

Productivity doesn't scale because the human becomes the limiting factor of the system. And the more you trust the agent, the more frustrating it becomes to have to review its work. It's like having a Ferrari and driving at 30 km/h.

## Phase 6: Automated Validation, Strategic Oversight

The final phase (for now) arrives when you understand that trust isn't built by reviewing the agent's work, but by building systems that validate the work automatically.

For code: tests that are generated and executed with every change. No passing tests, no integration. Validation is binary and automatic.

For thinking: reviewer agents. For research and planning, there are no tests to pass, but you can use a second agent with a critic role to review the first one's work. It's like automated code review applied to documents, plans, and architectural decisions.

You're no longer the filter. You're the architect of the validation system.

## And Phase 7...

I don't know that one yet. You'll probably discover it when you stop looking for it.

What I do know is that evolution isn't linear. You'll return to earlier phases with new perspective. Phase 6 isn't a destination, it's a starting point. And every qualitative leap in models will redefine what you thought you knew about programming with AI.

If you're in Phase 5, don't worry. Phase 6 exists. And Phase 7... that's built along the way.
