---
layout:     post
title:      "MiMo-V2.6-Flash on DeepSWE: 71 of 113 tasks, done in 90 steps per task"
subtitle:   "The same corpus, the same harness and the same task order as the GLM run: 62.8% resolved, 94.5% of target tests, and a different way to fail"
date:       2026-10-11 01:30:00 +0200
author:     "Daniel Vela"
locale:     en
---

This is the companion run to the GLM-5.3-Flash evaluation: same 113 tasks, same agent framework, same fixed task order, same binary grading. MiMo-V2.6-Flash-MOPD came out of it with **71 of 113 tasks resolved (62.8%)** and **94.52% of all target tests passing**, using markedly less effort per task than GLM — and failing in a way that is genuinely different.

## The setup

| | |
|---|---|
| Model | MiMo-V2.6-Flash-MOPD (Xiaomi), W4A8 (MXFP4 experts + FP8 dense) |
| Server | vLLM, tensor parallel 2 (2x RTX PRO 6000 Blackwell Max-Q) |
| Context | 448,512 tokens, KV cache in fp8, DFlash speculative decoding |
| Agent | mini-swe-agent, `model_class: litellm`, one task at a time |
| Per-task time budget | 180 minutes, the corpus's own limit |
| Task order | Fixed seed, identical to the GLM run |

Everything was served locally and quantized; the official board number for this model (about 68%) comes from a different setup with different weights, and I am not claiming to reproduce it.

## The result

| Metric | Value |
|---|---|
| Tasks resolved | **71 / 113 (62.8%)**, 95% CI [53.6 – 71.2] |
| Target tests passed (pooled) | 5,555 / 5,877 = **94.52%** |
| Target tests passed (mean per task) | 90.59% |
| Regression tests passed (pooled) | 231,316 / 231,352 = **99.984%** (36 failures) |
| Regression tests passed (mean per task) | 99.86% |
| Time per task | median 15.8 min, mean 18.1 min, max 46.3 min |
| Time spent | 31.4 h of agent time, 34.0 h of wall time |
| Effort | 10,190 steps (**90 per task**), far fewer than GLM's 139 |

The maximum is worth a sentence of its own: **the longest task took 46 minutes**, well under the corpus's three-hour ceiling. Whatever else this model does, it does not sprawl. It reached the same wall-clock average as GLM (18.1 vs 19.0 minutes per task) with about a third fewer steps, which says its steps are longer — more reasoning or more generation packed into each turn.

## Where the tasks land

![Tasks by share of target tests passed]({{ site.url }}/assets/benchmarks/deepswe/mimo-f2p-bands.svg)

The shape is similar to GLM's — most tasks either pass everything or land in the 90–99% band — but the tail is heavier. MiMo has **9 tasks below 50% of their target tests**, against 4 for GLM, and 7 in the 50–89% band. When this model misses, it misses further.

## By language

| Language | Resolved |
|---|---|
| Go | **26/35** |
| Python | 21/34 |
| TypeScript | 20/34 |
| Rust | 1/5 |
| JavaScript | **3/5** |

Go and JavaScript are where MiMo beats GLM by one task each. Python and TypeScript are where it loses ground, and Rust is a rout in the other direction: MiMo resolved one of the five Rust tasks, GLM resolved three.

## Where it earns its keep

Fourteen tasks were solved by MiMo and by nothing else in this comparison: `awilix-async-container-initialization`, `bandit-interprocedural-taint-checks`, `go-git-worktree-merge-conflicts`, `happy-dom-deterministic-intersectionobserver`, `ipython-session-bundle-replay`, `koota-pair-relation-tracking`, `mashumaro-flattened-dataclass-fields`, `onedump-dump-encryption-pipeline`, `prometheus-typed-label-sorting`, `scc-bounded-memory-spilling`, `superjson-error-stack-serialization`, `tengo-callable-instance-isolation`, `testem-per-launcher-reports` and `updo-policy-alerting`.

`mashumaro-flattened-dataclass-fields` is the cleanest example: MiMo scored 66/66 on the target tests, GLM scored 63/66. Three assertions, one point, and the two models are on opposite sides of the line. That is the sharpest illustration I have of how much the binary metric compresses.

## Its failure modes

Of the 42 failures:

- **23 near-misses** in the 90–99% band.
- **3 tasks with every target test passing and a regression costing the point**: `bandit-structured-nosec-directives`, `kombu-single-active-consumer-priority` and `vulture-persistent-analysis-cache` — the last one is also in GLM's list, so two different models broke the same existing behavior on the same task.
- **7 in the 50–89% band** and **9 below 50%**, the tail I mentioned above.

## MiMo against GLM, on the same 113 tasks

| | MiMo-V2.6-Flash-MOPD | GLM-5.3-Flash |
|---|---|---|
| Resolved | 71/113 (62.8%) | **76/113 (67.3%)** |
| Pooled target tests | 94.52% | **97.38%** |
| Mean per task | 90.59% | **94.71%** |
| Pooled regression tests | 99.984% (36 broken) | **99.993% (17 broken)** |
| Steps per task | **90** | 139 |
| Longest task | **46 min** | 79 min |
| Tasks under 50% of target tests | 9 | **4** |

GLM resolves more and gets closer on what it does not resolve; MiMo works more cheaply and has a narrower tail of catastrophic misses. Both statements are true at once, which is exactly the messiness the single pass-rate number hides — and the subject of the next post.

## What this does not prove

- **One run per task**, so the interval on 71/113 is [53.6, 71.2].
- **Quantized weights and a locally served endpoint**, not the configuration behind the published number.
- **Five Rust tasks.** The 1/5 here and the 3/5 for GLM are small-sample observations, not a ranking of the models on Rust.

## Reproducibility

The run is driven by the same `eval/` tooling described in the GLM post: a model registry entry with the serving metadata, a scheduler that survives being stopped, and one JSON record per batch plus a full agent trajectory and verifier log per trial. All 113 verifier logs for this model were audited for environment failures (network errors during dependency fetch, or a verifier that failed in both the base and new mode) and came back clean.
