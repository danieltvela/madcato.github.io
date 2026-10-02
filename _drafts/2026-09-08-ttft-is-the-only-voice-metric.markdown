---
layout:     post
title:      "TTFT Is the Only Voice Metric: Choosing an Inference Server for a Local Voice Assistant"
subtitle:   "Why aggregate throughput benchmarks don't predict how a voice assistant feels — and why the 'official' server of your platform might be the worst choice for your case"
date:       2026-09-08 00:00:00
author:     "Daniel Vela"
locale:     en
---

Most LLM server benchmarks answer the wrong question for a voice assistant. They measure aggregate throughput — how many tokens per second a box can push when a hundred requests hit it at once. A single-user voice assistant will never be that box. There is exactly one request in flight, and the human on the other end of the pipeline perceives exactly one number: **time to first token (TTFT)**. If the assistant takes two seconds to start talking, no decode speed can rescue the conversation. If it starts in under half a second, everything after feels instant.

This post is about choosing an inference server for a local voice assistant on Apple Silicon — and about an uncomfortable rule that came out of the exercise: **the "official" server of your platform's framework can be the worst option for your use case.** Mine was.

## The requirements that disqualify servers by themselves

Before any benchmark, there is a set of must-haves. A voice pipeline is not a chatbox: it acts. If the server cannot emit reliable tool calls over the wire, every turn where the assistant needs to *do* something dies silently.

For a server talking to an OpenAI-compatible Rust client, the non-negotiables are:

- **OpenAI-compatible SSE streaming**: `POST /v1/chat/completions` with `stream: true`, `data: {...}\n\n` chunks, `data: [DONE]` termination.
- **Native tool-calling in the stream**: `delta.tool_calls[]` carrying `function.name` and `function.arguments`, ending with `finish_reason: "tool_calls"`. Not a JSON blob in `delta.content` that you have to scrape with regex — the structured delta, or nothing.
- **Correct message roles** (`system`, `user`, `assistant`, `tool` with `tool_call_id`) so multi-turn history stays valid.
- **Per-request sampling parameters**, and both streaming and non-streaming paths (you stream conversation, you don't stream a summarization job).

Here is the uncomfortable part. The official `mlx-lm` server — the reference server shipped with the MLX framework, the natural default on Apple Silicon — openly acknowledges the gap. In its own discussions ([mlx-lm #371](https://github.com/ml-explore/mlx-lm/discussions/371)), the team describes tool calling as *"a fast moving target"* where the server *"aims to support common cases"*. Users reported Qwen3 tool-calling failures compared to GGUF/llama.cpp. For my project, where a broken tool call means the assistant simply cannot act, "common cases" is not a contract — it is a coin flip. And it is exactly the kind of limitation people accept by inertia because the server came from the right GitHub organization.

The same server also lacks automatic prefix caching between requests (only a basic rotating cache), which brings us to the real lever.

## The prefix cache is the lever, and nobody benchmarks it

A voice assistant re-sends a large, *unchanging* system prompt on every single turn — mine sits around 4–8K tokens. Time to first token is dominated by prefill, and on turn one, that system prompt is the prefill. On turn two, a server with a persistent prefix cache should not touch it again: it reuses the KV cache computed on turn one and only prefills the new user message.

The difference is not marginal. In my comparison, servers with automatic prefix caching (llama.cpp's `--cache-prompt`, on by default; vllm-mlx's hash-based prompt cache, always on) reduce turn-2+ TTFT by **10–30×** compared to re-prefilling the whole prompt every turn. An arXiv comparative study of LLM inference on Apple Silicon (2025) makes the flip side explicit: MLX requires full prefill before emitting — no cache, no mercy.

So the dominant latency lever for a multi-turn voice assistant is a feature that does not appear in a single number on any marketing benchmark, because benchmarks send one prompt and measure throughput. Meanwhile the "obvious" official server is the one that re-prefills your 8K system prompt every time you say hello.

## The recommendation is conditional on your chip

There is no universal winner, because the hardware changes the physics:

**M4 and earlier → `llama-server` (llama.cpp, Metal).** Contra Collective's 2026 benchmarks show llama.cpp beating MLX on prefill for an 8B-Q4 model on M4: **1,420 vs 1,180 tok/s** — Metal prefill optimizations winning on raw compute where Apple's Neural Accelerators don't exist yet. Add the battle-tested OpenAI tool-calling (works with Gemma, Qwen, Llama), `--cache-prompt` reusing the system prompt KV for free, MTP speculative decoding for MoE models baked into GGUF (~+12% decode), and the universal GGUF catalog. It is the boring, correct answer.

```bash
llama-server -m gemma-4-26b-a4b-MTP-Q4_K_M.gguf \
  --host 127.0.0.1 --port 8000 \
  --ctx-size 8192 --threads 8 --n-gpu-layers 99 \
  --cache-prompt --load-mode mmap+mlock \
  --spec-type draft-mtp --spec-draft-n-max 1
```

**M5 with ≥32 GB → `vllm-mlx` (excps fork).** On M5 the physics changes: Apple's own WWDC 2025 material puts the Neural Accelerators at roughly **4× TTFT advantage for MLX over Metal** on prefill — and that advantage exists *only* on M5. On top of that, vllm-mlx brings the two things mlx-lm should have had: a persistent hash-based prefix cache (always on in its SimpleEngine, no flags), and **17 tool-calling parsers with auto-recovery** — if the model emits malformed JSON for a tool call, the server repairs it instead of dropping the turn. Decode without speculation runs 43–74 tok/s on M3 Ultra-class hardware, slower than llama.cpp, but `--draft-model` closes the gap.

```bash
vllm-mlx serve mlx-community/gemma-4-26b-a4b-4bit \
  --host 127.0.0.1 --port 8000 \
  --tool-call-parser hermes \
  --draft-model mlx-community/gemma-4-1b-4bit \
  --num-draft-tokens 4
```

Same model class, different chip, different server. A recommendation that ignores your chip is marketing.

## What does NOT matter here

My other benchmark posts on this blog (the Qwen3 series, the FP8 quant study, the RTX PRO 6000 Blackwell vLLM recipe, the HermesAgent 2.0 numbers) all measure *models* on NVIDIA hardware with throughput-first metrics. That framing is useless here, and it is worth being explicit about why:

- **Continuous batching** is a multi-user feature. It overlaps prefill and decode across concurrent requests to maximize aggregate utilization. A voice assistant has one stream. The scheduler optimizations that make a benchmark chart look heroic add nothing you can hear. (One exception, noted in my own research: batching can slightly help *chained tool calls* within one turn — a rounding error compared to the cache lever.)
- **Aggregate tokens/second** is not the felt experience. You speak, one generation starts, and its TTFT plus a smooth decode above ~30 tok/s is the entire product.
- **Max context concurrency** — the subject of my in-progress post on the KV cache bottleneck — explains why vLLM caps *simultaneous requests* by memory. Here the problem inverts: the KV cache is not a ceiling to fight, it is a *asset to reuse* across turns. Different lever, same memory.

The general lesson: card-spec benchmarks predict server-farm economics, not conversation quality. Do not let a benchmark suite pick your voice assistant's server.

## Decide with data on YOUR machine: a 30-minute micro-benchmark

Public numbers rot. TTFT crossovers between runtimes change with every release and every model. The final call should come from a micro-benchmark against your own server, your own model, your own system prompt. Four scenarios cover it:

| Scenario | What it measures | Method |
|----------|------------------|--------|
| **A. Cold turn** | TTFT with full prefill: system prompt + history + new question | Time from POST to first `delta.content` in SSE. Repeat 3×. |
| **B. Turn 2 (cache hit)** | Whether the prefix cache is real | Second request, same server/session, same system prompt, new message. The number you actually live with. |
| **C. Tool-call** | Does `delta.tool_calls` + `finish_reason: "tool_calls"` actually arrive | Force a call ("What time is it?" → `current_time`) and inspect the SSE. PASS/FAIL, no partial credit. |
| **D. Non-streaming** | Latency of the summarization/extract path | `stream: false`, measure total. |

Order matters: cold first, warm second in the same session, or you measure a cache that was never populated. Then fill a matrix — each candidate server (llama-server, vllm-mlx, mlx-lm) on each machine — and decide:

1. Tool-call must PASS. It is a must-have, not a metric.
2. Lowest mean **warm** TTFT wins — turns 2+ are the overwhelming majority of a conversation.
3. Tie-break on cold TTFT.

The measurement loop is about twenty lines of Python: POST with `stream: true`, break on the first delta with `content`, subtract. Do it before you commit to a runtime for a year.

## One footnote: speculative decoding comes in two flavors

Both recommended servers support acceleration at decode time, but they are *not* the same mechanism and the flags do not transfer:

- **llama.cpp** uses **MTP** (multi-token prediction) heads *baked into the GGUF* (e.g. unsloth's MTP builds): `--spec-type draft-mtp --spec-draft-n-max 1`. No separate model, no shared-vocabulary problem. Without `--spec-type`, the default is `none` — you get nothing unless you ask.
- **vllm-mlx** uses an **external draft model**: `--draft-model <small model> --num-draft-tokens 4`. The draft must share tokenizer and vocabulary with the main model, and you pay its memory.

If you benchmark one flavor's production flags and deploy the other's defaults, your benchmark is fiction.

## The rule

For a single-user voice assistant: TTFT is the metric, the prefix cache is the lever, the chip picks the server, and the "official" server of your platform's framework is guilty until proven innocent. Prove it on your machine with scenario B — or move on.
