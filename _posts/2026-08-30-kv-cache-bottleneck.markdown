---
layout:     post
title:      "The KV Cache Bottleneck: Why `max-num-seqs` is not your concurrency limit"
subtitle:   "Understanding how memory constraints drive vLLM scheduling"
date:       2026-08-30 00:00:00
author:     "Daniel Vela"
locale:     en
---

If you have ever configured a vLLM server with a high `--max-num-seqs` (e.g., 128 or 256) and watched your logs as requests sit in `Waiting` while the engine only processes a handful of active requests, you have encountered the KV Cache bottleneck.

It is a common misconception that concurrency is a configuration setting. In reality, concurrency is a resource allocation problem.

## The Myth of `max-num-seqs`

In vLLM, the `--max-num-seqs` parameter is a **safety ceiling**, not a guaranteed capacity. 

Think of it as a limit on the scheduler's "attention span." It tells the engine: *"Do not allow more than X sequences to be active at once, even if you have infinite memory."* 

However, if you set `max-num-seqs=128` but your GPU memory only has enough room for the KV Cache of 5 concurrent requests, you will only ever see 5 requests in the `Running` state. The other 123 will stay in `Waiting`. The scheduler is not broken; it is protecting the system from an Out-of-Memory (OOM) error.

## The Real Constraint: KV Cache and PagedAttention

The true arbiter of concurrency is the **KV Cache**. To avoid re-computing the attention mechanism for every token, vLLM stores the Keys and Values in memory. Through **PagedAttention**, this memory is managed in fixed-size blocks, similar to how an operating system manages virtual memory.

The concurrency you can achieve is fundamentally tied to how many of these blocks you have available. The relationship is roughly:

$$\text{Max Concurrency} \approx \frac{\text{Total GPU Memory for KV Cache}}{\text{Average Sequence Length (Prompt + Generation)}}$$

### The "Prefill" Wall

One of the most critical behaviors to observe in your logs is the interaction between the **Decode** phase and the **Prefill** phase:

1. **Decode is greedy:** As running requests generate tokens, they continuously claim new blocks from the cache.
2. **Prefill requires commitment:** To move a request from `Waiting` to `Running`, the scheduler must ensure there are enough free blocks to accommodate the *entire* prompt (the prefill).

If your `GPU KV cache usage` is hitting 85-90%, the scheduler will often refuse to start any new prefills. This is why you might see `Avg prompt throughput: 0.0 tokens/s` in your logs. The engine isn't idle; it's working hard on the existing requests, but it's "too full" to admit anyone new.

## Lessons from the Logs

When monitoring a live deployment (for example, a Qwen-based service), watch for these diagnostic signals:

* **The Throughput Gap:** If `Avg generation throughput` is high but `Avg prompt throughput` is zero, you are memory-bound. The scheduler is waiting for existing sequences to finish and release blocks.
* **The Usage Threshold:** Once KV cache usage crosses the ~90% mark, the scheduler becomes highly selective. Beyond ~95%, you will likely see **preemption** (where the engine forcedly swaps out or recomputes a sequence to make room), which causes massive latency spikes.

## How to expand your capacity

If you need more concurrency, you cannot simply change a config flag. You must change your memory footprint:

1. **Quantize the KV Cache:** Moving from BF16 to **FP8** for the KV Cache is one of the most effective ways to double your effective token capacity without changing the model itself.
2. **Leverage Prefix Caching:** If your users send similar prompts, enabling prefix caching allows the engine to reuse existing blocks, making the "cost" of admitting a new request much lower.
3. **Manage Context Length:** Reducing the `max_model_len` directly reduces the maximum number of blocks any single request can claim, leaving more room for others.

***
