---
layout:     post
title:      "The Inverted Cost Model: Why Your Orchestrator Should Be the Cheapest Agent"
subtitle:   "Running a local 27B model as orchestrator cut my OpenCode sessions from $16.56 to $0.31 — 53x cheaper with better results"
date:       2026-07-09 08:00:00
author:     "Daniel Vela"
locale:     en
lang-ref:   inverted-cost-model-opencode
---

> I had my multi-agent cost strategy backwards. The orchestrator is your heaviest token consumer, so it should be the cheapest model — not the smartest one.

## The problem with the obvious approach

When I first started using orchestration plugins in OpenCode, my instinct was straightforward: use expensive, capable models for the main agents and reserve my local model for simple tasks like search or quick edits.

It makes sense on paper. The orchestrator coordinates everything, so it should be the best model, right?

Wrong. The orchestrator is the model that sees the most tokens. Every subagent response, every file read, every intermediate result passes through it. It consumes orders of magnitude more tokens than any specialist agent because it is always in the loop.

## The inverted cost model

The insight is simple: **run your heaviest token consumer locally, and use paid models only where they earn it.**

The orchestrator does not need deep reasoning. It needs to plan, delegate, track progress, and reconcile results. A 27B local model handles that surprisingly well. The agents that actually need capability — the ones doing deep analysis, architecture review, or documentation research — consume far fewer tokens because they are called selectively for specific tasks.

## Real numbers from a real session

I resolved a complex issue recently. Here are the actual metrics:

- **342 API calls**
- **36.7M tokens processed**
- **Total cost: $0.31**

The setup: OpenCode with Oh-My-Opencode-slim, running Qwen3.6-27B locally as the orchestrator and fixer. The specialist agents used GLM-5.2 (Oracle), DeepSeek-V4-Flash (Librarian, Explorer), and Kimi K2.7-Code (Designer) through OpenCode Go.

The same session with GLM-5.2 as the orchestrator would have cost **$16.56**.

That is a **53x cost reduction** for the same result. 95% of all tokens were processed locally at zero cost.

## How Oh-My-Opencode-slim makes this work

Oh-My-Opencode-slim is an agent orchestration plugin for OpenCode by Boring Dystopia Development. It defines six specialized agents, each with its own model assignment:

| Agent | Role | My model |
|-------|------|----------|
| **Orchestrator** | Planning, delegation, reconciliation | Qwen3.6-27B (local) |
| **Oracle** | Architecture review, deep reasoning | GLM-5.2 (OpenCode Go) |
| **Librarian** | Documentation, web research | DeepSeek-V4-Flash (OpenCode Go) |
| **Explorer** | Codebase search, grep | DeepSeek-V4-Flash (OpenCode Go) |
| **Designer** | UI/UX work | Kimi K2.7-Code (OpenCode Go) |
| **Fixer** | Focused implementation tasks | Qwen3.6-27B (local) |

The Orchestrator stays in the main session loop, coordinating work and delegating to specialists. The specialists run as background agents — they get their specific context, do the work, and return results. The Orchestrator never sees the full token payload of each specialist's work; it only sees the summary.

This is the key mechanism that makes the inverted model work: the orchestrator sees everything, but the specialists see only what they need.

## Why a local 27B model works as orchestrator

Qwen3.6-27B running on my RTX PRO 6000 (96 GB VRAM) via vLLM handles the orchestration role well for three reasons:

1. **Planning is not the same as reasoning.** The orchestrator's job is to decompose tasks, track state, and route work. It does not need to solve hard problems — it needs to coordinate people who do.

2. **Latency matters.** The orchestrator is called on every turn. A local model responds in milliseconds. A remote API call adds seconds of round-trip time, and that compounds across hundreds of calls in a session.

3. **Context window is sufficient.** The orchestrator needs to maintain session state, not reason about a full codebase. 262K context (reduced from 1M for stability) is more than enough.

## Adapting the opencode-go preset

Oh-My-Opencode-slim ships with an `opencode-go` preset designed for cost-efficient setups using OpenCode Go's low-cost models. The default configuration uses GLM-5.2 for the orchestrator, Qwen3.7-Max for Oracle, DeepSeek-V4-Flash for Librarian/Explorer/Fixer, and Kimi K2.7-Code for Designer.

My configuration adapts it by routing the orchestrator and fixer to my local model — the two biggest token consumers:

```json
{
  "opencode-go": {
    "orchestrator": { "model": "vllm-pro6000/Qwen3.6-27B", "skills": ["*"], "mcps": ["*", "!context7"] },
    "oracle": { "model": "opencode-go/glm-5.2", "skills": ["simplify"], "mcps": [] },
    "librarian": { "model": "opencode-go/deepseek-v4-flash", "mcps": ["websearch", "context7", "gh_grep"] },
    "explorer": { "model": "opencode-go/deepseek-v4-flash", "mcps": [] },
    "designer": { "model": "opencode-go/kimi-k2.7-code", "mcps": [] },
    "fixer": { "model": "vllm-pro6000/Qwen3.6-27B", "mcps": [] }
  }
}
```

The change is minimal — two model assignments swapped from remote to local. The effect is dramatic: the orchestrator alone accounts for the majority of token consumption in a session, so making it local eliminates the bulk of the cost.

If you do not have a local GPU, the same principle applies: use the cheapest capable model for the orchestrator. DeepSeek-V4-Flash through OpenCode Go is an excellent option — fast, cheap, and surprisingly competent at planning.

## The rule

**The orchestrator should be your cheapest, fastest model. The specialists should be your most capable ones.**

The orchestrator is a project manager, not an engineer. It delegates, tracks, and reconciles. The Oracle, Librarian, and Designer are the engineers who do the actual work. Give them the best tools. Give the project manager the cheapest desk.

## What I got wrong initially

My first setup used GLM-5.2 and DeepSeek-V4-Pro as the orchestrator with local models for simple tasks. It worked well but cost $0.71 per session. Flipping the model assignments — local orchestrator, paid specialists — brought it down to $0.31 while actually improving the workflow.

The orchestrator was the bottleneck all along. Not in capability, but in token volume. Every tool call, every file read, every agent response went through it. Making it local eliminated the cost entirely.

## Setup requirements

This approach needs:

- **A GPU that can run a 27B model.** My RTX PRO 6000 has 96 GB VRAM, but even a consumer GPU with 24 GB (like the 7900 XTX) can run a 27B model in Q4 quantization.
- **vLLM serving the model locally.** I run it via Docker with `gpu-memory-utilization` tuned to leave room for KV cache.
- **OpenCode with Oh-My-Opencode-slim installed.** The plugin handles all the orchestration logic — you just configure which model goes where.
- **OpenCode Go account** for the specialist models (pay-as-you-go, low-cost models).

## Links

- [OpenCode](https://opencode.ai) — The terminal-based coding agent
- [Oh-My-Opencode-slim](https://github.com/alvinunreal/oh-my-opencode-slim) — Multi-agent orchestration plugin (6.7k stars)
- [My previous OpenCode setup post](/2026/06/01/opencode-setup) — Full configuration walkthrough
- [RTX PRO 6000 Blackwell setup](/2026/06/18/max-blackwell-vllm) — Hardware and vLLM configuration