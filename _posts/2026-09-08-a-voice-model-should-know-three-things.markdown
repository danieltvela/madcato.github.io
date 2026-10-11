---
layout:     post
title:      "A Voice Model Should Know Three Things: The Narrow-Scope Assistant and Its Deep Worker"
subtitle:   "Why stuffing thirty tool definitions into a voice assistant's system prompt is the classic mistake — and what a single door plus a background worker does instead"
date:       2026-09-08 00:00:00
author:     "Daniel Vela"
locale:     en
---

The classic voice-assistant architecture has a seductive failure mode. You want the assistant to do more, so you add more tools to its system prompt. Shell, files, browser, calendar, MCP servers, three agents, a search stack — thirty tool definitions, four thousand tokens of instructions. The model gets *capable* on paper and sluggish in practice.

The costs are concrete, not aesthetic:

- **Every tool definition is prefilled into every turn.** In a voice pipeline, time-to-first-token is the whole experience. A fat, churning tools array taxes the very first number the human hears (and I have written separately about why TTFT is the only voice metric — the prefix cache you are blowing away is the lever).
- **Small models get confused by large tool inventories.** Voice loops run on small, fast models precisely because they have to answer in a human timeframe. Ask a 4B-parameter model to pick correctly among thirty near-identical schemas and you will hear it stall, misfire, or hallucinate arguments.
- **Every wrong tool choice is a spoken pause.** In a chat UI, a wasted round-trip is invisible. Out loud, it is an awkward silence the user attributes to you, the assistant, not to your prompt engineering.

So the design rule I landed on for Seneschal, my local voice assistant, sounds almost austere: the speech model knows three things — its own voice, when to answer, and that there is a door.

## The law: narrow scope

The project has a constitution (`CONSTITUTION.md`), and section II.9 is titled *Narrow Scope*. It reads, in part:

> Seneschal owns the **audio pipeline and conversational experience only**. Complex tasks (shell commands, file system, calendar, web navigation) are **delegated to external agents** via ACP (Agent Communication Protocol).

When the deep worker landed (issue #236, documented in `doc/deep-worker.md`), the principle got its operational phrasing: *el modelo de habla NO se satura de tools ni instrucciones; todo lo pesado va al worker* — the speech model is not saturated with tools or instructions; everything heavy goes to the worker.

The speech model's world is therefore tiny:

```
Voz → LLM de habla (sin thinking, pocas tools: voz + run_deep_task)
        └─ run_deep_task → DEEP WORKER (async, endpoint propio, thinking,
                            loop de tool-calling propio; pool: MCP, agentes,
                            ficheros, shell, pantalla, búsqueda)
```

It sees a handful of voice-shaped tools, plus exactly one door: `run_deep_task`. That is the entire capability surface of the model doing the talking. Everything else — MCP servers, files, shell, screen, search, nested agents — lives behind that door and never enters the voice model's context. The delegation decision is made by the speech model itself, with no intermediate router: it either answers, or it calls the door.

## The worker: its own endpoint, its own loop, its own brain

The deep worker is not a background thread of the voice model. It is a separate engine (`crates/seneschal-worker/`) with:

- **Its own LLM endpoint** — a different server, a different model, a different sampling configuration.
- **Thinking enabled.** This is the whole reason for two endpoints. Reasoning models are too slow for a voice loop: the speech pipeline already spends 180–400 ms just deciding the user finished speaking (that is Apple's on-device end-of-utterance latency, before the LLM sees a token), and a thinking pass on top of that destroys the turn. The worker has no such budget pressure — it can think as long as it needs to.
- **Its own tool-calling loop** — up to `max_iterations = 10` rounds of tool calls, capped by a `timeout_ms = 300000`. Ten iterations and five minutes, versus a spoken turn that must feel instant. Those are different orders of magnitude, which is exactly the point: they should not share a model.
- **Its own tool pool** — MCP servers, `read_file`/`write_file`/`list_dir`, `run_shell`, `take_screenshot`, `quick_search`/`deep_research`, plugins, and a nested `run_agent` tool that delegates further out to external agents (Hermes and friends) over ACP sessions.

The call flow from the voice side is deliberately one-sided: `run_deep_task` is marked `is_background`, so the pipeline emits an immediate spoken ack — *"estoy en ello"*, "I'm on it" — plays a processing sound, and persists only an **abbreviated** exchange into the conversation history. The user hears a promise, not a stall.

### Thinking without breaking the template

One genuinely awkward constraint shaped the worker: on MLX quantizations, thinking kwargs and tool definitions conflict in the chat template's Jinja — sending `chat_template_kwargs` for reasoning together with a `tools` array can break the render. Rather than fight it, the worker ships two thinking modes:

- **`external` (default):** two inferences. Step one plans *without tools*, with thinking on. Step two runs the tool loop *without thinking*, following the plan. Zero template risk.
- **`inline` (opt-in):** one pass sending thinking kwargs alongside tools — only for backends where that combination is verified to work (e.g., vLLM launched with `--enable-reasoning --reasoning-parser`).

Two-step planning-then-acting sounds like a workaround, and it is — but it is also a fine architecture in its own right: the expensive reasoning happens while no tools are in scope, and the cheap execution follows a written plan.

## Contracts that make the rewrite boring

The part I am proudest of is not the worker. It is what happens when you turn it off.

- **`DEEP_WORKER_ENABLED=0` is byte-for-byte the old system.** MCP servers, agents, and plugins register on the voice side, vision runs against the voice LLM, and `run_deep_task` does not exist. There is an E2E wiremock test named `e2e_deep_worker_disabled_is_transparent` to enforce exactly that.
- **Worker results never persist as voice tool-exchanges.** When the worker finishes, its answer arrives as an event with `tool_call_id: None`, which the main loop routes to the announcement window — one single spoken utterance. The speech model's history never contains the worker's ten iterations of tool-calls. The voice context stays lean not just on the way in, but on the way out.
- **Barge-in cancels the worker — except when it must not.** If the user interrupts mid-task, a `CancellationToken` aborts the loop and the cancellation is announced. But permission interjections from nested agents use a sentinel payload that the watcher deliberately ignores: cancelling there would kill the worker precisely while it waits for the user to answer "may I run this?" on behalf of a sub-agent. Knowing which interruptions are interruptions is a design decision, not a default.
- **The worker is inspectable by voice.** Saying "status" or "cancel" through `run_deep_task` queries or kills running tasks, and progress (plan, tool X, done) shows up as agent rows in the TUI and the control surface.

## The takeaway

Capability and context are different budgets, and the classic voice assistant spends them from the same wallet. The narrow-scope design splits the wallet: the talking model spends its context on *talking* — voice tools, timing, and one door — while a worker with a slower, heavier brain spends five minutes and ten tool loops on the actual errand, and hands back a single sentence.

A voice model should know three things: its voice, when to answer, and where the door is. Everything else is what's behind the door — and behind the door doesn't need to fit in the prefix.
