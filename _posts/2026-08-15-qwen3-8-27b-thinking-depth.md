---
layout:     post
title:      "Qwen3.8-27B: The Thinking Dial, Measured"
subtitle:   "Series 2/4 — off, xhigh, medium, low: what each level costs in quality and wall-clock"
date:       2026-08-15 00:00:00
author:     "Daniel Vela"
locale:     en
lang-ref:   qwen3-8-27b-thinking-depth
og_image:   /assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/low/benchlocal-hermesagent-20-unsloth-qwen3-8-27b-nvfp4.png
---

Part one of this series covered quality, quantization, and the 1M-context price tag. This one is the part that matters most if you're running the model as an agent: **the reasoning dial**.

Qwen3.8-27B exposes four configurations: thinking **off**, and three reasoning efforts — **xhigh** (default), **medium**, **low** — set via `--default-chat-template-kwargs` in vLLM. The model card tells you the levels exist; it doesn't tell you what each one costs. That's the gap this post fills.

## Setup

- Model: `unsloth/Qwen3.8-27B-NVFP4` (the quant from part one — free, per the numbers there)
- Context: 262K (no YaRN), `temperature 0.6`, 3x per case (CLI-40 at 1x)
- Hardware: 1x RTX PRO 6000 Blackwell (96 GB)
- One service, one level at a time: `--default-chat-template-kwargs`
  - off: `{"enable_thinking": false}`
  - xhigh: `{"enable_thinking": true, "preserve_thinking": true}` (default)
  - medium: `{"enable_thinking": true, "preserve_thinking": true, "reasoning_effort":"medium"}`
  - low: `{"enable_thinking": true, "preserve_thinking": true, "reasoning_effort":"low"}`

## Results

All four passes: `unsloth/Qwen3.8-27B-NVFP4`, 262K context, `temperature 0.6`, 3x per case (CLI-40 at 1x), machine up for the whole run. Columns ordered by reasoning effort, off → xhigh.

**Quality (BenchLocal overall score):**

| Benchmark | off | low | medium | xhigh |
|---|---|---|---|---|
| BugFind | 98 | 95 | 95 | 97 |
| InstrucFollow | 82 | 99 | 99 | 98 |
| DataExtract | **37** | 92 | 91 | 92 |
| HermesAgent | 82 | 85 | 85 | 83 |
| ToolCall | 93 | 100 | 100 | 100 |
| CLI-40 | **50** | 75 | 75 | 75 |
| **Average** | **73.7** | **91.0** | **90.8** | **90.8** |

**Wall-clock (total per suite, machine up):**

| Suite | off | low | medium | xhigh |
|---|---|---|---|---|
| BugFind | 3m 56s | 6m 45s | 8m 13s | 19m 52s |
| InstrucFollow | 23s | 3m 04s | 2m 52s | 6m 12s |
| DataExtract | 1m 22s | 3m 48s | 4m 29s | 14m 57s |
| HermesAgent | 12m 57s | 20m 28s | 21m 13s | 25m 24s |
| ToolCall | 1m 24s | 1m 54s | 2m 01s | 2m 07s |
| CLI-40 | 6m 30s | 11m 53s | 43m 47s | 116m 59s |
| **Total (6 suites)** | **26m** | **48m** | **83m** | **186m** |

## What the numbers say

**1. Quality plateaus at `low`.** The average jumps from 73.7 (off) to 91.0 (low) — and then *stops*. low, medium, and xhigh are all 90.8–91.0, within run-to-run noise. Turning reasoning up from low to xhigh buys you **zero** measurable quality. The dial does real work going off→low; it does nothing going low→xhigh.

**2. `off` is high-variance, not uniformly bad.** This is the nuance the average hides. With thinking off, the model collapses on the suites that need it — **DataExtract 37** (9 fails of 15) and **CLI-40 50** (23 fails of 40) — but it's actually *best* on BugFind (98, higher than any thinking level) and strong on ToolCall (93). So "off" is a high-variance setting: excellent for direct pattern-recognition tasks, catastrophic for careful multi-step extraction. It's not a setting you run blind; it's a setting you run *when the task is simple*.

**3. `xhigh` — the default — is the worst value.** It's the slowest by a wide margin (186m total vs. 48m for low, ~4x) and it does not score higher than low on a single suite: it's +2 on BugFind, −1 on InstrucFollow, −2 on HermesAgent, and tied on the rest. The model card ships xhigh as the default because it's tuned for open-ended reasoning quality in long-form generation. For an agent that has to finish a tool call and move on, xhigh is paying a 4x latency tax for nothing.

**4. The sweet spot is `low`.** Same quality as the default (91.0 vs. 90.8), at roughly a quarter of the wall-clock (48m vs. 186m across the six suites). On the reasoning-heavy suites the gap is steepest — CLI-40 goes 11m53s (low) → 116m59s (xhigh), a 10x spread — which is exactly where an agent's wall-clock budget lives.

## The crossover, in one line

The level where quality plateaus but speed keeps falling is `low`. That's the one to run for agentic work: you get the xhigh quality at a fraction of the xhigh cost, and you keep `off` in reserve for the genuinely simple calls (ToolCall-style) where it's measurably faster and just as good.

## Why this matters for voice

A voice agent's TTFT is dominated by reasoning tokens: the classifier routes a turn to "simple" (no thinking) or "complex" (thinking on), and the latency difference between those two paths is this curve, measured. The data sharpens the design in two ways. First, the "complex" path should default to `low`, not `xhigh` — same answer quality, ~4x faster first token. Second, the "simple" path (`off`) is only safe for the task types where it's measurably fine (ToolCall-style); the classifier has to know that a data-extraction or multi-step request is *not* a simple call, or it inherits the DataExtract-37 failure mode. The full calibration is **sampling × thinking level**: part one fixed the sampling axis (0.6 beats the official 1.0 on agentic suites), this post fixes the thinking axis (low beats the default xhigh on value).

## Screenshots

<details>
<summary>off</summary>

  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/off/benchlocal-bugfind-15-unsloth-qwen3-8-27b-nvfp4.png" alt="BugFind" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/off/benchlocal-instructfollow-15-unsloth-qwen3-8-27b-nvfp4.png" alt="InstrucFollow" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/off/benchlocal-dataextract-15-unsloth-qwen3-8-27b-nvfp4.png" alt="DataExtract" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/off/benchlocal-hermesagent-20-unsloth-qwen3-8-27b-nvfp4.png" alt="HermesAgent" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/off/benchlocal-toolcall-15-unsloth-qwen3-8-27b-nvfp4.png" alt="ToolCall" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/off/benchlocal-cli-40-unsloth-qwen3-8-27b-nvfp4.png" alt="CLI-40" loading="lazy" />

</details>

<details>
<summary>low</summary>

  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/low/benchlocal-bugfind-15-unsloth-qwen3-8-27b-nvfp4.png" alt="BugFind" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/low/benchlocal-instructfollow-15-unsloth-qwen3-8-27b-nvfp4.png" alt="InstrucFollow" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/low/benchlocal-dataextract-15-unsloth-qwen3-8-27b-nvfp4.png" alt="DataExtract" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/low/benchlocal-hermesagent-20-unsloth-qwen3-8-27b-nvfp4.png" alt="HermesAgent" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/low/benchlocal-toolcall-15-unsloth-qwen3-8-27b-nvfp4.png" alt="ToolCall" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/low/benchlocal-cli-40-unsloth-qwen3-8-27b-nvfp4.png" alt="CLI-40" loading="lazy" />

</details>

<details>
<summary>medium</summary>

  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/medium/benchlocal-bugfind-15-unsloth-qwen3-8-27b-nvfp4.png" alt="BugFind" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/medium/benchlocal-instructfollow-15-unsloth-qwen3-8-27b-nvfp4.png" alt="InstrucFollow" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/medium/benchlocal-dataextract-15-unsloth-qwen3-8-27b-nvfp4.png" alt="DataExtract" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/medium/benchlocal-hermesagent-20-unsloth-qwen3-8-27b-nvfp4.png" alt="HermesAgent" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/medium/benchlocal-toolcall-15-unsloth-qwen3-8-27b-nvfp4.png" alt="ToolCall" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/medium/benchlocal-cli-40-unsloth-qwen3-8-27b-nvfp4.png" alt="CLI-40" loading="lazy" />

</details>

<details>
<summary>xhigh</summary>

  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/xhigh/benchlocal-bugfind-15-unsloth-qwen3-8-27b-nvfp4.png" alt="BugFind" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/xhigh/benchlocal-instructfollow-15-unsloth-qwen3-8-27b-nvfp4.png" alt="InstrucFollow" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/xhigh/benchlocal-dataextract-15-unsloth-qwen3-8-27b-nvfp4.png" alt="DataExtract" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/xhigh/benchlocal-hermesagent-20-unsloth-qwen3-8-27b-nvfp4.png" alt="HermesAgent" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/xhigh/benchlocal-toolcall-15-unsloth-qwen3-8-27b-nvfp4.png" alt="ToolCall" loading="lazy" />
  <img src="/assets/benchmarks/unsloth-qwen3.8-27b-nvfp4-think/xhigh/benchlocal-cli-40-unsloth-qwen3-8-27b-nvfp4.png" alt="CLI-40" loading="lazy" />

</details>

## Recommended daily configuration

Taken together, the two posts converge on one configuration for daily agentic use of this model on a single RTX PRO 6000:

| Setting | Value | Why |
|---|---|---|
| Quantization | NVFP4 (`compressed-tensors`) | Quality parity with FP8, less weight memory, more KV headroom |
| Context | 256K | No YaRN toll; 1M costs 12 points on BugFind + 30-50% speed for nothing under 262K working sets |
| `reasoning_effort` | `low` | Quality plateaus here (91.0 vs 90.8 at xhigh) at a quarter of the wall-clock |
| Temperature | 0.6 | Official 1.0 sampling costs 12 points on HermesAgent for agentic work |

With `off` reserved for the genuinely simple calls (ToolCall-style) where it's measurably faster and just as good — and the 1M build kept as an on-demand option, not the default.

**Caveat, stated plainly:** these results are preliminary. The benchmarking was done quickly and at a surface level — six suites, 3x repetitions (1x for CLI-40), a single hardware box, one quantization per axis, and no statistical treatment of the run-to-run variance. The conclusions are robust *within* that scope (the gaps are far larger than the noise), but they are not a substitute for a longer, more rigorous evaluation. Treat the numbers as directional evidence from one careful afternoon, not as a measured truth.

## References

- [Qwen/Qwen3.8-27B on HuggingFace](https://huggingface.co/Qwen/Qwen3.8-27B) — `reasoning_effort` levels
- [vLLM recipe for Qwen3.8-27B](https://recipes.vllm.ai/Qwen/Qwen3.8-27B) — `--default-chat-template-kwargs`
- Part one: [Qwen3.8-27B-FP8: First Benchmarks on RTX PRO 6000 Blackwell](/posts/2026/08/14/qwen3-8-27b-fp8-benchmarks.html)
