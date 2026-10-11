---
layout:     post
title:      "Write the Constitution Before the Code: Governing a Project That Agents Build"
subtitle:   "When the implementer does not remember yesterday, invariant rules and dated decision records are the real human-owned asset"
date:       2026-09-08 00:00:00
author:     "Daniel Vela"
locale:     en
---

Most advice about AI-assisted development is about process: how to prompt, how to decompose tasks, which agent goes first. I wrote some of that myself. But process answers *how* agents should work. It does not answer the question that actually decides whether a project survives a year of agent sessions: *what rules must not change while the workforce keeps turning over?*

In Seneschal, my Rust voice assistant, the majority of the code has been written by agents. What I own is not the code. It is two documents: a constitution, and a stack of dated decision records. That turn out to be the load-bearing part.

## A constitution, literally

The repo has a file called `CONSTITUTION.md`. Its opening is not decorative:

> This document defines the **invariant rules** and **design principles** that govern the Seneschal project. Every decision — architectural, code-level, operational — must conform to these principles. Amendments require explicit deliberation; no agent or contributor may override them implicitly.

It is written the way an engineering contract should be: rules plus rationale, not vibes. A few examples, verbatim from the document:

- **Single Binary.** "All pipeline stages run in-process connected by `tokio` channels. There is no inter-service communication, no microservices, no IPC." The rationale is stated right below it: no serialization overhead, no network round-trips, trivial cancellation propagation, easy to reason about.
- **No Speculative LLM on Local GPU.** The Apple Silicon GPU is reserved for STT and TTS; running a speculative LLM on the same GPU "would cause contention and jitter," so LLM inference is delegated to an external process.
- **Legacy Modules (do not extend).** A concrete list — a deprecated Whisper wrapper, an obsolete WebSocket client, an unused Python provider — with the standing order: "Do not add features to legacy modules. Flag for removal if found."

Each rule closes a door an agent would otherwise happily reopen. "No microservices" is not a stylistic preference; it is the difference between a project that stays a single understandable binary and one where a well-meaning refactor spins the pipeline into four containers talking gRPC to each other. Agents are excellent at doing plausible things. The constitution is the list of plausible things that are forbidden here.

## The amendment procedure as an access-control mechanism

Section VII of the document is the part most people would skip, and it is the part that makes the rest real. To change any principle you must:

1. State the principle to be changed.
2. Explain the rationale.
3. Identify affected modules and migration path.
4. Be reviewed against all other principles for consistency.

And the closing sentence: "No agent may amend this document autonomously."

Read that as what it is: an access-control list on the highest-privilege file in the repo. An agent can propose, implement, refactor, and rewrite — but the layer that defines what "correct architecture" means is writable only through human deliberation. In political theory terms that I find weirdly precise here: the human holds the constituent power, the agents hold the constituted power. They operate inside the rules; they do not silently rewrite them by operating.

The quiet killer this prevents is *implicit* amendment. An agent does not edit `CONSTITUTION.md`. It just adds an IPC boundary "temporarily", or reintroduces a banned module as a dependency, and next session the new agent reads the code as the source of truth and the drift becomes policy. Requiring an explicit, structured amendment forces the change into the open, where it either gets ratified or gets rejected — and either outcome is recorded.

## The stress test: the project outgrew itself

Constitutions look like ceremony until a real crisis tests them. Ours came in July. By then the project had outgrown its original size — roughly 25,000 lines of Rust, ~110 source files, ~30 docs, with subsystems (MCP, multi-agent delegation, plugins, a control API, a TUI) that were useful but tangled with the voice core.

The natural agent-era answer is "rewrite it." We did the opposite, and the record of that choice is a dated file, `CARVEOUT-DECISIONS.md`, headed:

> Origin: direct user instruction — project has outgrown the original goal (~25k LOC, ~110 `.rs`, ~30 docs). Direction: **salvage inventory + carve-out**, keep extended vision with strict module boundaries.

It is a decision record in the old ADR sense, but exhaustive. Every module in `src/` got a verdict — SALVAR, AISLAR, or DESCARTAR (keep, isolate, discard) — with a destination crate and a stated reason. The discards are as explicit as the keeps: an embedding classifier whose feature was empty in `Cargo.toml`, a Piper wrapper never declared in the module tree, a stale `.bak` file in the source tree. Then a target workspace layout, a dependency matrix showing `seneschal-core` as a leaf crate that depends on nothing else in the workspace, and a decisions log. The first entry in that log: "Doc first, code second — reconcile contradictions and fill gap docs before touching source."

Only after the document was finished did the code move. Result: 13 out of 13 crates carved out, all changes `git mv` only — no logic rewrites — with the QA harness green. A ~25k-LOC reorganization executed by agents in a matter of days, without a rewrite, because the *decision* was made once, in writing, and the agents did exactly what a written decision makes possible: mechanical, verifiable, boring work.

## Why this transfers to any agent-built project

The structural fact of agentic development is recruitment turnover at machine speed. Every session, a new worker enlists who has read the codebase but not lived it. It knows what the code says; it does not know why three options were tried and two were buried. Conventions rot in that soil, because nobody alive remembers the convention's rationale.

The constitution is institutional memory. It answers, at the moment the agent is about to type, the question the previous session cannot: *is this allowed?* The decision records answer the second one: *what did we already decide, and when?* Together they are the two things a workforce with no long-term memory needs — a bill of rights and an archive.

Most projects already have the first ingredient in embryo: `AGENTS.md`, the now-ubiquitous instructions file. It is worth being clear about the gap. An AGENTS.md typically carries commands, a build matrix, an architecture sketch, a docs index — Seneschal's does exactly that, including a pointer to run the QA harness before any PR. Useful? Genuinely. But it says *here is how the project works*, not *here is what must never change, and here is the only procedure by which it may*. It has no amendment clause, no precedence over the code, no notion of a change being illegal rather than merely unusual. The constitution is what you get when you take the good instinct behind AGENTS.md and give it teeth: inviolability, a defined change process, and dated records instead of folklore.

## Where it hurts

I would be overselling the mechanism if I skipped the failure mode: constitutions rot if they are not cited, and rules written in a calm month can be contradicted by a noisy market a season later.

There is a live example in my own repo. The constitution says "Apple MLX Backends Only" for LLM inference. But the provider evaluation doc, written later after benchmarking TTFT across servers, concludes that the *current* server — official mlx-lm — "has critical limitations in tool-calling and prefix cache that make it unreliable for this project," and formally lists it under discarded options with "Migration recommended," pointing instead at `llama-server` for older chips and a forked `vllm-mlx` for the newest ones.

That is a contradiction between the constitution and reality. And here is the thing: registering the tension *is part of the mechanism, not a bug in the document*. The constitution stays citable precisely because the disagreement happened in a dated doc that names the tension, instead of happening in code, where it would have been invisible drift. The amendment exists to be invoked — a contradiction sitting in writing forces the human deliberation the whole system is designed to require. What would be a bug is pretending the contradiction isn't there because the founding document said so.

## The human-owned asset

If agents write most of your code, your leverage moves. It moves away from typing and toward deciding: which invariants are worth freezing, which door you will regret leaving open at 2 a.m. six months from now, and what you actually decided the week the system was still small enough to remember.

Write those down before the code needs them. Give them a change procedure you control. Date every decision you make under pressure. The implementers will not remember yesterday — but the constitution does, and it does not ask for a salary.
