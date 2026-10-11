---
layout:     post
title:      "GLM-5.3-Flash on DeepSWE: 76 of 113 tasks, and 25 more that were one test short"
subtitle:   "A local 4-bit GLM-5.3-Flash through the whole DeepSWE v1.1 corpus: 67.3% resolved, 97.4% of all target tests passing, and a failure profile that says more than the score"
date:       2026-10-11 01:00:00 +0200
author:     "Daniel Vela"
locale:     en
---

I ran the whole DeepSWE v1.1 corpus against GLM-5.3-Flash, quantized to four bits and served locally on two consumer Blackwell cards. Not a subset, not a smoke test: all 113 tasks, one at a time, each with the benchmark's own hidden test suite grading the result. This is what came out.

Short version: **76 of 113 tasks resolved (67.3%)**, **97.38% of all target tests passing**, and 17 broken tests out of 231,352 regression tests. The number I keep coming back to is different though: 25 tasks scored zero while passing 90–99% of their target tests.

## What the benchmark is, and how it grades

DeepSWE v1.1 is a corpus of 113 tasks, each derived from a real pull request in a real repository: a base commit, an instruction, and a hidden test suite. The suite splits in two. The **fail-to-pass (f2p)** tests are the ones the change is supposed to make pass; the **pass-to-pass (p2p)** tests are the repository's existing tests, which must keep passing.

The grading is binary per task: **a task scores 1 only if every f2p test passes and no p2p test fails**. Anything else is a zero. There is a partial metric in the harness, but it is p2p-dominated and I will not use it; what I report below is the binary pass rate plus the graded share of target tests, which I compute myself.

The agent sees only the instruction and the repository at the base commit. The tests live in a separate container that the agent never touches, and the agent runs with no network access. It cannot see what it is being scored on.

## The setup

| | |
|---|---|
| Model | GLM-5.3-Flash (Zhipu AI), W4A16 NVFP4 experts + FP8 weights |
| Server | SGLang, tensor parallel 2 (2x RTX PRO 6000 Blackwell Max-Q) |
| Context | 524,288 tokens, KV cache in fp8, EAGLE speculative decoding |
| Agent | mini-swe-agent, `model_class: litellm`, one task at a time |
| Per-task time budget | 180 minutes, the corpus's own `[agent] timeout_sec` |
| Task order | Fixed seed, identical to the run I did with MiMo, so comparisons are task by task |

Everything ran unattended across four nights, resuming from a state file after each interruption. Wall clock is not the interesting part; I mention it because a 113-task run only becomes practical if it survives being stopped.

## The result

| Metric | Value |
|---|---|
| Tasks resolved | **76 / 113 (67.3%)**, 95% CI [58.2 – 75.2] |
| Target tests passed (pooled) | 5,723 / 5,877 = **97.38%** |
| Target tests passed (mean per task) | 94.71% |
| Regression tests passed (pooled) | 231,335 / 231,352 = **99.993%** (17 failures) |
| Regression tests passed (mean per task) | 99.74% |
| Time per task | median 16.3 min, mean 19.0 min, max 78.6 min |
| Time spent | 32.9 h of agent time, 35.8 h of wall time |
| Effort | 15,654 steps (139 per task), ~99k output tokens per task, 64% of them reasoning |

Two ways of aggregating appear above on purpose. Pooled means "of all the target tests in the corpus, what share passed"; the per-task mean means "in a typical task, what share of its tests passed". They differ because the tasks are wildly unequal in size — from 2 to 254 target tests — and I will come back to that in a later post, because it is the most interesting thing I found in this whole exercise.

## Where the tasks land

![Tasks by share of target tests passed]({{ site.url }}/assets/benchmarks/deepswe/glm-f2p-bands.svg)

Read the second bar. **Twenty-five tasks passed 90–99% of their target tests and scored zero.** They are not failures in any useful sense of the word: they are changes that got one edge case wrong, or left one branch untested, out of suites that often contain more than a hundred assertions.

The other end is thin: only 4 tasks ended below 50% of their target tests. This is not a model that collapses on hard work; it is a model that consistently gets most of the way and then misses a detail.

## By language

| Language | Resolved | Notes |
|---|---|---|
| Go | 25/35 | the largest group; failures spread thin |
| Python | 25/34 | |
| TypeScript | 21/34 | |
| Rust | 3/5 | all five Rust tasks are in the corpus |
| JavaScript | 2/5 | |

Rust is the group I care about most and the group the benchmark covers least — five tasks is a sample, not a verdict. On those five, GLM resolved three, a better rate than any other model I have run locally.

## Three ways to lose a point

The 37 failures break down into three distinct stories, and only one of them is a real "the model could not do this" story.

**Three tasks where every target test passed and a regression cost the point.** `gql-incremental-graphql-delivery` scored 17/17 on target tests and 810/811 on regression; `helm-array-merge-strategies` scored 47/47 and 10/12; `vulture-persistent-analysis-cache` scored 24/24 and 291/295. The feature is implemented; something that used to work stopped working. If I were shipping this code I would want exactly that information, and the binary metric does convey it — but it conveys it identically to a task where nothing works at all.

**Twenty-five near-misses** in the 90–99% band, plus five more in the 50–89% band.

**Four genuinely far off**: `boa-hierarchical-evaluation-cancellation` (0/17 target tests), `eicrud-keyset-pagination-cursor`, `kgateway-consistent-hash-policy` (0/2) and `onedump-dump-encryption-pipeline`. For what it is worth, MiMo fails the first two as well.

## What I take from it

GLM-5.3-Flash is a *mostly right* model, and DeepSWE's binary metric punishes exactly that shape. Ninety-seven percent of all the target tests in the corpus pass; more than two thirds of the tasks are resolved outright; and a quarter of the corpus sits one test away from being counted. If your question is "can this model do the work", the pooled number is the honest answer. If your question is "will it ship without a regression", the binary number is the honest answer, and it is 67.3%.

The one weakness I would flag for real use: GLM's failures concentrate in regressions on existing behavior rather than in the requested behavior. That is the kind of error a code review catches and a test suite catches better.

## What this does not prove

- **One run per task.** No repetitions. The 95% confidence interval on 76/113 is [58.2, 75.2] — wide, because 113 binary outcomes carry only so much information.
- **Quantized weights.** This is a 4-bit build (NVFP4 experts, FP8 weights), not the full-precision model the official leaderboard measured at ~63%. Quantization is part of the result, not a footnote to it.
- **My harness is not the official one.** Same corpus, same grading container, same agent framework, but my own driver, my own serving stack, and locally served weights. Numbers will differ from the board's, which for GLM-5.3-Flash is around 63%.
- **Rust is five tasks.** Anything I say about Rust is a five-task observation.

## Reproducibility

The run lives in a fork of the DeepSWE repository with an `eval/` directory that drives it: `eval/models.yaml` for the model registry (identity, quantization, serving metadata), `eval/nightly.py` for the scheduler, and a results tree with one JSON record per batch. Every trial keeps its own container logs, the agent's full trajectory, and the verifier output, so a number can be traced back to the run that produced it. All 113 verifier logs for this model were audited for environment failures — network errors during the build, or a verifier that failed in both the base and the new mode — and none were found.

The aggregate report is an HTML page generated from those records; the numbers above come from it, cross-checked against the raw trial files.
