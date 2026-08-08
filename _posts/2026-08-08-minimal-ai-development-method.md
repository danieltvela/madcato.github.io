---
layout:     post
title:      "A Minimal Method for AI-Assisted Software Development"
subtitle:   "Why the two popular approaches fail, and what to use instead"
date:       2026-08-08 08:00:00
author:     "Daniel Vela"
locale:     en
lang-ref:   minimal-ai-development-method
---

There are two dominant ways people use AI to develop software right now. Both have obvious appeal. Both are traps — not because the ideas are wrong, but because they solve the wrong problem.

## The two approaches that don't work

### Approach 1: Role-based agents with Git

You set up a team of AI agents — PM, Architect, Senior Dev, Junior Dev, QA, DevOps — and manage the project through GitHub or Gitea. Issues, branches, PRs, code review. The human stays in the loop at every step.

The appeal is that it mirrors how human teams work. The fatal flaw is that it replicates the *process* of human development without recognizing that the process exists to compensate for human limitations. An AI agent doesn't need a PM to write requirements it can read directly. It doesn't need a QA to test what it can verify itself. It doesn't need a DevOps engineer to manage branches when it can do it in one command. The human in the loop becomes a bottleneck, not a contributor.

The deeper problem: this approach optimizes for *familiarity*, not for *efficiency*. You use the same workflow that human teams use because it's what you know. But a workflow designed for slow, error-prone communicators is not a workflow you should give to an entity that reads a codebase in seconds and never gets tired.

### Approach 2: Full orchestration with specialized agents

Frameworks like Oh-My-OpenAgent spawn multiple agents that talk to each other, pass context, spawn sub-agents, and coordinate autonomously. The user gives a high-level instruction and the system builds everything.

The fatal flaw is token economics. Every agent-to-agent message is a token cost. Every sub-agent is another model call. The orchestrator sees every intermediate result, reproceses it, and passes it on. The token count explodes. You can offset this with free models, but that thinking is backwards: optimizing cost should be the *design constraint*, not a patch applied after the architecture is set. A system that burns 500K tokens to do what 50K tokens could do is a bad system regardless of whether the tokens are free.

The subtler problem: orchestration frameworks create the illusion of intelligence by multiplying agents. More agents feels like more capability. But the bottleneck in AI-assisted development is not the number of agents — it is the quality of the specification and the verification of the output. More agents just means more token waste between agents that are all trying to solve the same problem from different angles.

## The third path: one agent, three files, automated Git

The method I've been converging on has three components. It is not simpler because it cuts features. It is simpler because it eliminates everything that doesn't contribute to the output.

### 1. The Blueprint Trio

Every project starts with three files that completely define what the project is and how it will be built. These are not documentation — they are the interface between human intent and machine execution.

#### SPEC.md — The contract

The SPEC defines what the project is and what it is not. It contains:

- **Objective**: a one-sentence description of the project's purpose.
- **Non-objectives**: explicit scope exclusions. What this project will not do. This is as important as the objective — it prevents feature creep and gives the agent clear boundaries.
- **Core concepts**: the key data models, architectural components, and API contracts. The agent should be able to understand the project's structure without asking questions.
- **User flow**: the step-by-step interaction model. How does a user (or another system) interact with the project?
- **Acceptance criteria**: a checklist of "done" states. Each criterion must be binary (pass/fail) and verifiable (a command, a test, an observable behavior).

The SPEC is the single source of truth. No agent ever needs to ask "what are we building?" because the answer is in this file. The SPEC also serves the human: if you cannot write a clear SPEC, you do not have a clear plan, and no amount of agent orchestration fixes that.

#### TASKS.md — The execution plan

The TASKS file is a sequential, phased roadmap. Each phase groups related work. Each task within a phase is atomic — it identifies the target file(s) and a specific action. Vague tasks like "implement authentication" are forbidden. Instead: "Add `authenticate()` method to `src/auth.rs` that validates credentials against the `users` table and returns a JWT token."

Each phase ends with a **verification block**: a command that must pass, a test that must succeed, or a behavior that must be observable. No phase is complete until verification is proven. This is not optional — it is the mechanism that replaces a QA role.

The TASKS file also includes a **"No touching"** section for each phase: files or areas that must not be modified. This prevents the agent from "helpfully" refactoring unrelated code to meet its own aesthetic preferences.

#### CONSTITUTION.md — The law

The Constitution contains hard rules that no agent can bypass. It includes:

- **Hard constraints**: rules about payload integrity, security, or architectural choices. For example: "No `unsafe` Rust without a written justification in the commit message." Or: "All user input must be validated before processing."
- **Verification protocol**: rules on how tasks are closed. For example: "The agent must run the project's test suite and all tests must pass before marking a task complete."
- **Escalation triggers**: explicit conditions for when the agent must stop and mark a task as blocked for human review. For example: "If a task requires a dependency not in the project's lock file, stop and report." Or: "If a test fails three consecutive times, stop and report the error."

The Constitution prevails. The agent cannot "suggest" an exception to speed things up. If the Constitution says no, the answer is no.

### 2. A single orchestrator agent

Not a team. One agent. It reads the SPEC, works through the TASKS in order, and stops when verification fails or the Constitution is violated. It handles Git operations — creating branches, making commits, opening PRs — without human intervention. The human reviews the final result, not the process.

The orchestrator doesn't need to be the smartest model. Planning, delegating, and tracking progress are not deep-reasoning tasks. A local model with sufficient context handles the orchestration role well. Expensive models are called only when the orchestrator identifies a task that needs deep analysis or architectural reasoning — and even then, only for that specific task, not for the entire session.

The key insight from the inverted cost model: the orchestrator is the highest token consumer in the system. Making it the cheapest capable model eliminates the bulk of the cost. The specialists (when used at all) are called sparingly and only for tasks that genuinely need their capability.

### 3. Phased execution with verification gates

Each phase in TASKS.md has a verification block. The agent cannot advance to the next phase until the current phase's verification passes. This eliminates the need for a separate QA role — verification is built into the execution flow.

If a test fails, the agent fixes the issue and re-runs the verification. It does not move forward until it passes. If it hits an escalation trigger, it stops and reports. The human decides whether to adjust the Constitution, modify the SPEC, or intervene directly.

This is fundamentally different from the "fire and forget" approach of full orchestration. The agent has guardrails, not freedom. It can work autonomously within well-defined boundaries, but it cannot drift.

## Why this works

**No redundant communication.** A team of agents passes context between each other, reprocesing the same information at every hop. The orchestrator + SPEC has zero inter-agent communication overhead. The agent reads the spec once, executes, verifies. The SPEC is the context. It does not need to be passed around.

**No man-in-the-loop for routine work.** The human doesn't approve branches or review PRs for routine changes. They review the final state. This is the difference between "I need to check every step" and "I need to check the result." The human is the reviewer, not the gatekeeper.

**Cost is a first-class design constraint.** The cheapest model handles the orchestrator (the highest token consumer). Paid models are reserved for tasks that genuinely need their capability. This is not a cost optimization applied after the fact — it is the architecture. The method starts with the constraint and builds from there.

**The method is project-agnostic.** SPEC + TASKS + CONSTITUTION works for a Rust TUI, a Python web service, a Swift iOS app, or a static site. The format is the same. Only the content changes. The method does not care what language, framework, or platform you are building on.

**The method scales with complexity.** A small project might have one phase with three tasks. A large project might have ten phases with fifty tasks. The structure is the same. The SPEC, TASKS, and Constitution grow with the project, but the workflow does not change.

## What this is not

This is not fully autonomous software engineering. The human defines the project in the SPEC, sets the Constitution, and reviews the final result. The agent handles execution. The division is: human defines *what*, agent handles *how*.

This is not a replacement for thinking about your project. The SPEC forces you to be precise about what you are building. If you cannot write a clear SPEC, you do not have a clear plan — and no amount of agent orchestration fixes that.

This is not "let the AI figure it out." The Constitution exists precisely to prevent the agent from taking shortcuts that violate the project's integrity. The agent works within constraints, not around them.

## Getting started

1. Create `SPEC.md`, `TASKS.md`, and `CONSTITUTION.md` for your project.
2. Start the orchestrator with the SPEC loaded.
3. Let it work through TASKS.md phase by phase.
4. Review the result. Iterate on the SPEC if the result doesn't match your intent.

The hardest part is writing a good SPEC. Everything else follows from that. A bad SPEC produces a bad result regardless of how good the agent is. A good SPEC produces a good result even with a mediocre agent.

## Links

- [The AI Project Blueprinting skill](/hermes/skills/ai-project-blueprinting) — the formal methodology behind the Trio
- [The Inverted Cost Model](/2026/07/09/inverted-cost-model-opencode) — how to route models by role for minimum cost