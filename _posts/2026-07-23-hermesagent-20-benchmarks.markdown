---
layout:     post
title:      "Comparing LLMs on the HermesAgent-20 Benchmark"
subtitle:   "Local testing of agentic capabilities on RTX PRO 6000 Blackwell using BenchLocal.app"
date:       2026-07-23 08:00:00
author:     "Daniel Vela"
locale:     en
lang-ref:   inverted-cost-model-opencode
og_image:   /assets/benchmarks/hermes/benchlocal-hermesagent-20-gemma-4-31b-it-nvfp4.png
---

I've been testing several high-performance models on the **HermesAgent-20** benchmark suite via [BenchLocal.app](https://benchlocal.com). This specific benchmark is designed to evaluate the "agentic" qualities of a model: its ability to follow complex multi-step instructions, maintain state, and orchestrate tool calls effectively within a simulated agent environment.

All tests were performed on my RTX PRO 6000 Blackwell (96 GB VRAM) to ensure consistent hardware performance across the board.

## The Contenders

I selected a mix of dense and MoE models, all utilizing NVFP4 quantization to maximize throughput and VRAM efficiency.

### Gemma 4-31B-IT-NVFP4
Google's latest iteration. A very balanced model that often punches above its weight in reasoning and instruction following.

![Gemma 4]({{ site.url }}/assets/benchmarks/hermes/benchlocal-hermesagent-20-gemma-4-31b-it-nvfp4.png)

### NVIDIA Qwen3.6-27B-NVFP4
The current workhorse of my local setup. Known for extreme efficiency and high reliability in coding and orchestration tasks.

![Qwen3.6]({{ site.url }}/assets/benchmarks/hermes/benchlocal-hermesagent-20-nvidia-qwen3-6-27b-nvfp4.png)

### NVIDIA Nemotron-3-Super-120B-A12B-NVFP4
A heavyweight MoE model. With 120B total parameters but a smaller active set, it aims to bring "frontier" reasoning to local hardware.

![Nemotron-3]({{ site.url }}/assets/benchmarks/hermes/benchlocal-hermesagent-20-nvidia-nvidia-nemotron-3-super-120b-a12b-nvfp4.png)

### Poolside Laguna-S-2.1-NVFP4
The latest from Poolside, specifically optimized for coding agent workflows. It has shown impressive results in SWE-bench and Terminal-Bench.

![Laguna S 2.1]({{ site.url }}/assets/benchmarks/hermes/benchlocal-hermesagent-20-poolside-laguna-s-2-1-nvfp4.png)

## Observations

The HermesAgent-20 results provide a glimpse into how these models handle the "cognitive load" of being a primary orchestrator. While raw reasoning (MMLU/GSM8K) is important, agentic benchmarks reveal the gaps in *reliability*—the tendency to drop a tool call or lose track of a goal over a 20-turn interaction.

For those interested in the exact scores, the screenshots above contain the full BenchLocal breakdown including Pass/Partial/Fail counts.

## Links

- [BenchLocal.app](https://benchlocal.com) — The benchmark suite used for these tests.
- [The Inverted Cost Model](/2026/07/09/inverted-cost-model-opencode) — Why I prioritize efficiency for my orchestrator.
- [Maximizing the RTX PRO 6000](/2026/06/18/max-blackwell-vllm) — Hardware and vLLM configuration for these runs.
