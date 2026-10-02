---
layout:     post
title:      "The Model Still Wins: A Black-Box Benchmark of a Tuned Harness vs a Stronger Model"
subtitle:   "Two agent stacks, one frozen HTTP contract and 50 hidden tests: the harness bought speed and structure, the model bought correctness"
date:       2026-10-02 00:00:00
author:     "Daniel Vela"
locale:     en
---

Everyone building agent tooling shares a quiet hope: that enough instructions, skills and custom tools can compensate for a weaker model. I wanted to test that hope instead of assuming it. So I built a small, deliberately boring benchmark: two complete agent stacks, one task, one frozen contract, and a hidden test suite that neither stack could see. The result surprised me less than the *shape* of the failures.

Short version: the stronger model on the plain harness scored 47/50 (94%) in 23 minutes; the weaker model on the fully tuned harness scored 38/50 (76%) in 11m46s. The harness nearly doubled the throughput of the weaker stack — and did not buy a single error-path behavior.

## The task: a contract you cannot argue with

The task is a Vapor 4 REST API in Swift, on Linux, with no Xcode: a small library-lending service called *shelf*. Users register and log in (JWT HS256, 3600 s tokens), admins manage books, members borrow and return them, fines accrue at 50 cents per overdue day, and all state lives in a single SQLite file. The prompt specifies the exact HTTP contract: endpoints, status codes, JSON shapes, ten stable error codes (`validation_failed`, `email_taken`, `no_copies`, `loan_limit`, `already_returned`, `book_has_open_loans`, `copies_below_open_loans`...), and even a `X-Test-Date` header so due dates and fines can be tested deterministically without touching the clock.

That rigidity is the point. The more the deliverable is defined as observable HTTP behavior, the less a grader has to trust — or even read — the generated code.

## The method

The benchmark is a measurement instrument, so the method matters more than the blog post. These are the decisions I would defend:

**Freeze the tests before the first run, and derive them from the contract.** The hidden suite was written from the prompt alone, never from a finished solution, and it never entered either agent's workspace. The prompt and the suite are hashed together (`02d225c0…`); if the hash changes, the old results are void.

**Grade the process, not the code.** The 50 tests are pure HTTP against a running server: no imports, no static analysis, no reading the package. The test runner is Python standard library only, on purpose — zero dependencies, nothing to install, impossible to accidentally couple to the implementation.

**Two phases, because persistence is part of the contract.** Phase A runs 45 tests against a fresh server with default environment. Then the same SQLite file is restarted with a custom `PORT` and a different `JWT_SECRET`: phase B checks that state survives, that old tokens are rejected, and that environment variables are honored. A stub that builds but never serves fails cleanly (`startup_failed`), and an empty package fails as `build_failed`.

**Calibrate against a reference solution.** Before scoring anything I wrote a reference implementation and required `task/grade.sh task/reference` to report 50/50. That calibration caught two suite bugs (a list/object assertion and a key-presence check) and one real contract ambiguity before it could poison a comparison. The same command is re-run whenever the suite changes.

**Isolate the arms from the repository.** Both workspaces live *outside* the benchmark repo, seeded only with the task prompt plus that arm's files. The control arm is a plain `pi -p --no-session --no-context-files --no-skills` invocation, so even a stray `AGENTS.md` in a parent directory cannot leak into it. The tuned arm gets its pack copied into the workspace.

**Same grading environment for both.** Build and server run in a pinned `swift:6.1-jammy` image; the test runner runs in `python:3.12-slim`. Runs are launched detached (Herdr panes) with the exact canonical command, and the launcher records harness versions, the suite hash, the treatment hash and wall-clock timestamps next to the artifact.

## The two stacks

|  | Arm B (control) | Arm A (tuned) |
|---|---|---|
| Model | GLM-5.3-Flash (locally served, SGLang) | Gemma-4 31B (locally served, vLLM) |
| Harness | pi 1.0.0, plain: no context files, no skills | OpenCode 1.18.34 |
| Instructions | none beyond the prompt | `AGENTS.md` + project config + skills (`swiftpm-linux`, `vapor-service`, `fluent-sqlite`) + commands (`/verify`, `/smoke`) + a custom `swift` tool that wraps Docker |

The two models are both locally served and GPU-exclusive, so the arms ran sequentially: B first, then A after a model swap. Arm A's "optimization" is exactly what a developer would do for a real Swift project: tell the agent where the toolchain is, how to build in Docker, which Vapor idioms to use, and give it buttons for the repetitive verification steps.

## The result

| Arm | Hidden tests | Wall time | Build |
|---|---|---|---|
| B — GLM-5.3-Flash + plain pi | **47/50 (94%)** | 23m00s | ok |
| A — Gemma-4 + OpenCode pack | **38/50 (76%)** | 11m46s | ok |

Both stacks produced complete, buildable projects: package manifest, models, migrations, controllers, Dockerfile, README. Both agents also claimed success in their final messages. Both were wrong in ways only the hidden suite could see.

## Forensics: where the 15 points went

The failure lists are more instructive than the scores.

**The failure both stacks shared.** Both omitted the `returnedOn: null` key from loan responses. Swift's synthesized `Codable` omits nil optionals unless you write a custom `encode(to:)`, so `returnedOn` simply disappears from the JSON — while the contract explicitly writes `returnedOn=null`. Arm A's `vapor-service` skill even warned about this exact trap. Gemma read the warning and walked into it anyway. That is a good reminder that a skill is guidance, not a guardrail: only the test is a guardrail.

**Arm A's cluster: error shapes.** Ten of Gemma's twelve failures are error-path failures: an expired JWT, a token signed with the wrong secret, a malformed token, a body missing a required field, a wrong password — all came back as 500 with `internal_error` when the contract says 401 or 400 with a stable code; the `/stats` route returned a `RouteNotFound` wrapped as a 500 to members. The remaining two failures are the shared `returnedOn` omission. The app's happy paths were largely fine; its error paths were an afterthought.

**Arm B's anomaly.** GLM's three failures include the same nil-omission and one oddity: a book payload missing `title` returned 401 instead of 400, as if authentication was evaluated where validation should have been. I cannot fully explain it from the outside, which is exactly what black-box grading is for.

**The self-test illusion.** Arm B wrote its own `integration-test.sh` with 106 checks and its final report proudly claimed all of them passing. It still scored 94%. Arm A's summary claimed "builds and passes all tests" at 76%. Agents write tests that confirm what they built; a contract suite tests what was specified. Those sets overlap less than anyone would like.

## What the harness bought, and what it did not

What it bought: **speed and structure**. Arm A finished in half the time and used the custom `swift` tool and the `/verify` loop visibly: write controllers, build in the pinned image, fix Fluent key-path errors, repeat. For a weaker model, offloading the infrastructure knowledge (where is Swift, how to build on Linux, what does a Vapor app look like) removed an entire class of distractions. The pack also produced a slightly more conventional layout (controllers, middleware, services).

What it did not buy: **behavior at the edges**. Every failure that mattered was in an error path — precisely the place where a model's discipline shows and instructions are easiest to ignore. If anything, the tuned stack's failures were more clustered and systematic (one broken error-mapping decision repeated everywhere), which suggests the pack should ship a *verification* for error envelopes, not just prose about them.

## Three infrastructure lessons

**Reasoning models can spend the entire output budget thinking.** The first control run died after two minutes with an empty workspace and exit code 0. The logs showed the model had burned all 16k output tokens on reasoning and hit the length limit (`stopReason: length`); pi treated the truncated turn as settled and exited successfully. The fix was not in the prompt but in the harness config: raise the output budget to 65536 on both stacks. If I had been measuring "quality" without checking usage, I would have measured the cap, not the model.

**A benchmark run can fail silently and look fine.** Exit code 0, a clean shell prompt, and a log with one warning line — that was a failed run. Progress in this kind of experiment is file creation and GPU activity, not exit codes. The launcher now records `started_at`, `finished_at`, `exit_code` and the harness log in an adjacent `.bench/` directory, and a run is only "done" when the marker and the artifact agree.

**Local model servers are exclusive, and the schedule has to respect it.** Two stacks meant two model servers, each using both GPUs. The arms ran sequentially, and the whole comparison took one afternoon of wall time — fine for a case study, but the reason repetitions are expensive.

## What this does not prove

This is one task, one run per arm, with model *and* harness deliberately confounded, because the question was "which complete stack delivers better software", not "how much does each factor contribute". To separate them you would need a 2×2: both models on both harnesses, several repetitions per cell, and a second task with a different failure profile. What I can say is narrower and still useful: on this contract, with these exact stacks, the stronger model on the plain harness won on correctness, and the tuned harness made the weaker model competitive on time, not on points.

## What I would change next

- Repetitions: at least three per arm per task, reporting mean and spread of the pass rate, not a single number.
- A second and third task, chosen to stress different contracts (pagination semantics, relational integrity, idempotency) rather than the same REST shape.
- Aggregate tokens and cost from the harness logs, not just wall time.
- Add error-envelope tests as a first-class section of the skills pack, with a command the agent can run against its own server. The data says that is where the tuned stack lost.
- Record the suite hash in every published comparison. It is the only way a reader can tell whether two numbers were measured with the same ruler.

## Reproducibility

The repository is [danieltvela/bench-gemmaopencode-glmpi](https://github.com/danieltvela/bench-gemmaopencode-glmpi): the frozen prompt (`task/prompt.md`), the hidden suite (`task/hidden_tests/`, 50 tests in two phases), the reference solution used for calibration (`task/reference/`), the grader (`task/grade.sh`), the launcher (`task/run-arm.sh`) and the recorded results (`results/run1-*.json`, `results/runs.csv`). The suite hash for these numbers is `02d225c0…`; arm A's treatment hash at run time was `6802dd39…`. Both arms built, booted, and were graded by the same commands in the same pinned images.

The boring conclusion is the useful one: **a harness can be optimized, but correctness has to be verified** — by tests the agent did not write, from a contract it cannot renegotiate.
