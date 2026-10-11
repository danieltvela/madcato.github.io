---
layout:     post
title:      "Four local models, five Rust tasks: a per-task comparison"
subtitle:   "MiMo, GLM, Qwen3.8-Flash-Next and Laguna-S-2.1 on the smallest language group of DeepSWE — including a task that needed 75 minutes, a model that ran out of context, and a verifier that lied"
date:       2026-10-10 19:00:00 +0200
author:     "Daniel Vela"
locale:     en
---

DeepSWE covers five languages, and Rust is the smallest slice of it: five tasks out of 113. That makes it a bad sample and an irresistible one — Rust is what I actually care about, compiling is where verification gets expensive, and five tasks are few enough to compare four models task by task instead of by average.

So that is what this is. The same five tasks, four locally served models, the same agent framework, the same grading container, one run each. Not a leaderboard; a close reading.

## The four models

| | MiMo-V2.6-Flash-MOPD | GLM-5.3-Flash | Qwen3.8-Flash-Next | Laguna-S-2.1 |
|---|---|---|---|---|
| Lab | Xiaomi | Zhipu AI | Alibaba | Poolside |
| Quantization | W4A8 | W4A16 NVFP4 + FP8 WO | NVFP4 | NVFP4 |
| Server | vLLM | SGLang | vLLM | vLLM |
| Context | 448,512 | 524,288 | **262,144** | **1,048,576** |
| Tensor parallel | 2 | 2 | **1** | 2 |

All four ran on the same two RTX PRO 6000 Blackwell cards, one model at a time, because a locally served model owns both GPUs. Qwen is the odd one out twice over: the shortest context window by a factor of two, and the only one that fits on a single card.

## The five tasks

Straight from the corpus: `boa-hierarchical-evaluation-cancellation` (the Boa JavaScript engine), `fd-deterministic-multi-key-sorting` (the fd file finder), `oxvg-structural-selector-preservation` (an SVG optimizer), `pest-character-class-coalescing` (the pest parser generator) and `wasmi-trap-coredumps` (the wasmi WebAssembly interpreter).

| Task | MiMo | GLM | Qwen | Laguna |
|---|---|---|---|---|
| `boa-hierarchical-evaluation-cancellation` | 0/17 | 0/17 | 0/17 | 0/17 |
| `fd-deterministic-multi-key-sorting` | **43/43 ✓** | **43/43 ✓** | **43/43 ✓** | 42/43 |
| `oxvg-structural-selector-preservation` | 4/6 | 4/6 | **6/6 ✓** | 6/6 (p2p 60/62) |
| `pest-character-class-coalescing` | 100/104 | **104/104 ✓** | 102/104 | 82/104 |
| `wasmi-trap-coredumps` | 20/22 | **22/22 ✓** | 0/22 | 8/22 |

✓ marks a resolved task: every target test passing and no regression. Numbers are target tests passed out of the suite size.

![Share of target tests passed across the five Rust tasks, for four local models]({{ site.url }}/assets/benchmarks/deepswe/rust-four-models.svg)

| | Resolved | Target tests (mean per task) | Regression tests | Total time | Steps per task |
|---|---|---|---|---|---|
| GLM-5.3-Flash | **3/5** | 73.3% | 100.00% | 2.71 h | 246 |
| Qwen3.8-Flash-Next | 2/5 | 59.6% | 99.92% | **1.57 h** | 142 |
| MiMo-V2.6-Flash-MOPD | 1/5 | 70.7% | 99.92% | 1.86 h | 129 |
| Laguna-S-2.1 | 0/5 | 62.6% | 99.27% | 3.03 h | **365** |

## Four stories, one per task

**`boa` is the wall.** None of the four models passed a single one of its 17 target tests, and none of them broke anything either (7/7 regression tests, all four). This is the task that tells me the suite in this corpus is not decorative: it is a change inside a JavaScript engine's evaluation machinery, and every local model I have tried bounces off it.

**`fd` is the floor.** All four models implemented deterministic multi-key sorting in the file finder. Laguna came within one assertion (42/43); the other three were perfect. If a task in this group is going to be solved, it is this one — and the interesting signal is not the score but the cost: MiMo took 11 minutes and GLM 11 minutes, Qwen 13, Laguna 19.

**`oxvg` is the discriminator.** Qwen and Laguna implemented all six target behaviors; MiMo and GLM stopped at four. But the two that "solved" it did not get the same result — Laguna's patch passed 6/6 target tests and broke two existing ones (60/62), so it scores zero. This is the cleanest example in the whole comparison of a model doing the requested work and losing the point on a regression.

**`wasmi` is where effort and context collide.** GLM resolved it (22/22) in 26 minutes. MiMo got 20/22. Laguna got 8/22. And Qwen did not get evaluated at all: its agent's prompt reached **263,213 tokens against a 262,144-token window**, the call failed with a context-length error, the agent exited before committing, and the harness collected an empty patch. The verifier then graded the pristine repository and reported 0/22 — which the binary metric duly recorded as a failure, indistinguishable from a model that never understood the task.

That last one deserves the emphasis. With a 448k or 524k window, MiMo's and GLM's own peak contexts on this task (326,215 and 339,270 tokens) would have overflowed Qwen's window too. The 0/22 is a property of the serving configuration as much as of the model, and nothing in the score says so.

## The verifier that lied

Laguna's first pass on `oxvg` came back as **0/6 target tests and 0/62 regression tests** — a catastrophic result that, on inspection, had nothing to do with the model. The verifier log showed both the base run and the new run failing with `rc=102` and 64 lines of `Could not resolve host: static.crates.io`: the verification container could not download Rust dependencies, so it never compiled the pristine repository either, and with no JUnit output the harness counts every test as failed.

The task was re-queued and re-run. The second pass had a healthy verifier and produced the 6/6 with two regressions reported above. The difference between the two results is not a model difference; it is a network difference.

That cost me an afternoon, so I turned it into a routine: every verifier log in a run is scanned for exactly this pattern — network failures, or a build that failed in both modes. Across all the runs in this project (194 verifier logs: 113 MiMo, 71 GLM at the time, 5 Qwen, 5 Laguna), **exactly one was invalid**, and it is the one above. The check is a `grep`, which is the right price for not publishing a fake zero.

## What I take from five tasks

- **Ownership is per task, not per model.** Each of the five has a different answer: one nobody solves, one everybody solves, one only Qwen and Laguna touch, one only GLM, one only GLM with everyone else failing differently. Averaging these into a single "Rust score" throws away the only interesting part.
- **Context length is a functional parameter, not a spec sheet number.** The cheapest model in this group to run was also the only one whose run was cut short by its own window.
- **The measurement instrument needs auditing as much as the thing being measured.** A verification container that cannot reach a package registry produces zeros that look exactly like failures.
- **Compiled languages are where the harness gets expensive.** These five tasks took 1.6 to 3.0 hours each in total, and Rust verification means compiling a repository twice (base and patched) per task. That cost is why this group has five tasks and not fifty.

## What this does not prove

Five tasks, one run each, four models. The confidence intervals on 3/5, 2/5, 1/5 and 0/5 are enormous and overlapping. Two of the four models (Qwen, Laguna) have been run on Rust only, so nothing here says anything about how they behave on the other 108 tasks. Treat this as a case study of four specific models on five specific changes, not as a ranking.

## Reproducibility

Each model has its own state file and its own run records; the Rust subset was selected by language from the same corpus manifest that drives the full runs, in the same seeded order, so the task-by-task comparison is exact. Every trial keeps the agent's full trajectory, the container logs and the verifier output, including the failed first pass of `oxvg` for Laguna, which is where the `static.crates.io` evidence lives.
