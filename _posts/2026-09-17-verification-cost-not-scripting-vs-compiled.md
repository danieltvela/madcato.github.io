---
layout:     post
title:      "The axis isn't scripting vs compiled: it's the cost of verification"
subtitle:   "Languages for AI agents: what the August 2026 debate actually settled"
date:       2026-09-17 22:20:00
author:     "Daniel Vela"
og_image:   "/img/post-bg-01.jpg"
locale:     en
lang-ref:   verification-cost-not-scripting-vs-compiled
---

My whole career I've carried a balance that seemed permanent: scripting languages (Ruby, Python, JavaScript) buy you iteration speed and pay for it with errors that only surface at runtime; compiled languages (C, Java, Rust) buy you guarantees and pay with a slow loop — you wait for the build to find out if anything works. Xcode Previews was the most serious attempt to give the compiled world the scripting loop: change how your UI looks without compiling or launching the debugger. When it works, it's a delight. When it doesn't, you go back to waiting.

With agentic programming the suspicion is natural: if an agent lives by iterating, scripting languages should win hands down, because their verification loop is instant. I suspected it too. After following the actual 2026 debate, my conclusion is different: **the scripting/compiled dichotomy is the wrong framing**. The right axis is the cost of verification — and that axis split the dichotomy in half.

## The debate got formalized in August 2026

On August 11, the Go team published [Why Go is an Ideal Language for AI-Assisted Software Engineering](https://developers.googleblog.com/why-go-is-an-ideal-language-for-ai-assisted-software-engineering/). The premise, which almost nobody disputes: when an agent generates hundreds of lines in seconds, **the bottleneck stops being writing code and becomes verifying it**. All the useful analysis of 2026 revolves around that shift.

Google's argument: `gofmt` gives you total uniformity (generated code reads like human code), the static type system kills hallucinated APIs at compile time and — the decisive detail — Go compiles orders of magnitude faster than Java, C# or Rust, so the agent runs generate → compile → fix → recompile without waiting. Their number (stated, not publicly measured): agents produce valid Go on the first pass 95% of the time.

Hacker News pushed back hard across hundreds of comments: no benchmarks, some said; and Uber's data-race audit (46 million lines, 2,000+ races, 790 patches) disqualifies the thesis, said others. The comment that won the debate for me pointed somewhere else entirely: a fussy compiler is *ideal* for an LLM — hammering it with tokens is a far better strategy than trying to deduce where things will fail at runtime and catching that with tests. Tokens are cheap; runtime surprises are not.

## The counter-thesis that undoes pure scripting

My hypothesis assumed waiting was the only cost of compiled languages. There's another one, inverted: the compiler is the best hallucination detector that exists, and scripting delegates all of that to the runtime.

The academic benchmark that the whole 2026 guardrail ecosystem cites ([arXiv:2404.00971](https://arxiv.org/abs/2404.00971)) found that 15.1% of code hallucinations are of the "API that doesn't exist or was never imported" kind — and that **less than 10% of hallucinated code fails the tests**. Translation: in a scripting language, most hallucinated code sails through the test suite and only blows up on the execution path nobody exercised. The compiler kills entire *classes* of error on every pass, for free; the runtime only kills *instances*, only on executed paths. For an agent generating thousands of lines per session, that coverage gap is enormous.

On top of that, the error signal matters as much as its existence: a compile error is deterministic, precise and free for the model to interpret. A runtime stack trace only reports what was executed. The compiler-driven correction loop is more *informative* per iteration.

## The empirical cold water

What nobody on either side wants to hear: recent benchmarks say the language matters less than both camps believe.

- **MirrorCode** (2026), on long-horizon tasks: small differences in solve rates across Python, C, Rust, Go, OCaml and Ada; only modest token differences between successful runs. Current models **transfer their capability across languages** better than people assume. ([summary of the debate](http://hndebrief.com/2026-08-11/whats-the-best-programming-language-for-coding-agents))
- The danluu.com benchmark (agents implementing a Zstandard decoder and a Pandoc-like converter in several languages) was received with healthy skepticism: the community steered the conversation back to tooling, verification and conventions, and left one piece of advice that survives the noise: optimize your repo and workflows around verification loops; treat per-language token efficiency as a weak signal.
- The observation I consider the most important one: **architecture matters more than language purity**. A cleanly componentized system with sharp contracts makes a "weak" language easier for an agent than a monolith in a strict one.

## My ledger: three factors, three winners

To evaluate a language as an agentic work surface:

    1. Cost of one self-correction round-trip     → scripting (Python/JS) and Go
    2. Error classes eliminated by verification   → Rust (and to a lesser degree TS/Go)
    3. Human review load and uniformity           → Go, typed TypeScript

The "scripting always" position optimizes factor 1 and forgets 2. The "Rust is better, it just is" position optimizes 2 and forgets 1. Both are incomplete for the same reason: the axis is verification, not the execution model.

And the champion of the latency argument was not Python: it was Go, a compiled language. Go is the proof that the dichotomy was split in half from the inside: static types + sub-second compilation.

## The convergence dissolving the debate

The industry is resolving the question by hybridization, not by victory:

- **TypeScript** is today's dominant agentic language for UI, and it is exactly that hybrid: scripting runtime + instant compiler (tsc). The Xcode Previews case already happened on the web years ago: hot reload (Vite/HMR) moves verification inside the loop.
- **Python** answered the same way: pyright/mypy + ruff work as "a compiler you can invoke in milliseconds" on top of a scripting runtime. Serious agentic repos run the type check as part of the agent loop, the same way they run tests.
- **Armin Ronacher** ([Fast and Hard Code](https://lucumr.pocoo.org/2026/8/22/fast-hard-code/), August 2026) — normally on the scripting side — predicts the opposite of my initial suspicion: *more* software in statically typed compiled languages, because agents absorb the Rust/Zig learning curve that used to hold adoption back. Cloudflare is building services in pure Zig, Vercel shipped a coding agent written in Zig; nearly all of it, per him, LLM-assisted.
- Two anecdotes pointing in opposite directions, both real: OpenAI migrated its Codex CLI from Rust to Python in 2026 (from 648K LOC down to 41K, a 15.9× reduction; [arXiv:2604.11518](https://arxiv.org/abs/2604.11518)) — the build tax and Rust's ownership model were killing iteration. In the opposite direction, porting scikit-learn to Rust with agents turned out to be viable with current models (Max Woolf, February 2026).

In one sentence: **the market is turning every popular language into "scripting with instant verification"**. The dichotomy dissolves into a different question: how cheap and how complete is verification?

## Practical verdict

- **Python**: the most versatile for agents (ML ecosystem, instant loop). Condition: pyright/ruff/tests as part of the loop, not an afterthought.
- **TypeScript**: the de facto standard for agentic UI — the previews case solved for years now.
- **Go**: the best backend compromise — the cheapest self-correction loop in the compiled world, uniform human review.
- **Rust**: when entire classes of bugs must die at compile time (concurrency, safety). Agents handle it well now; the loop at scale is still slow.
- **Avoid for agent-first work**: heavy JVM/Maven builds and giant C++ monorepos — the build-plant tax is real.

And the ground rule, the one I believe survives this debate: **choose by cost of verification × coverage of error classes × quality of the error signal — not by execution model**. With models that transfer capability across languages, what makes the real difference is not the language but the verification infrastructure you put around it: invokable types, tests, linters — the exact equivalent of Xcode Previews, but for logic.
