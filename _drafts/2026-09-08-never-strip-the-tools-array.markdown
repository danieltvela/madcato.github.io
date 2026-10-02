---
layout:     post
title:      "Never Strip the Tools Array: A Cheap Intent Router for Local Voice Assistants"
subtitle:   "Classification can be cheap; the shape of your inference request cannot"
date:       2026-09-08 00:00:00
author:     "Daniel Vela"
locale:     en
---

A local voice assistant faces a small, annoying asymmetry on every single turn. Half of what you say to it is "hola", "gracias", "vale" — turns that need a warm, casual sentence and nothing else. The other half is "investiga esto", "configura lo otro" — turns that need tools, low temperature, and structure. If you serve every turn with the same heavy configuration, you pay for it twice: robotic, tool-primed responses to your greetings, and a steady tax of tool-schema tokens and spurious tool-call attempts on turns that needed none of it.

So you build a router. A cheap one. The interesting part is not the router — it is the one thing the router is *not* allowed to touch.

## The cascade

Seneschal's classifier (`src/classifier/`) is a four-level cascade, each level cheaper than the last one is smart:

**Nivel 1 — heuristic (always on).** Deterministic, zero-cost, and embarrassingly effective for a voice assistant:

- Empty text or a trivial greeting (`hola`, `buenos días`, `gracias`, `vale`, `ok` — a hardcoded list of 17 Spanish patterns, matched after cleaning and lowercasing) → `Simple`, confidence 1.0.
- A match against any of **23 default Spanish complex keywords** — `investiga`, `lanza`, `ejecuta`, `analiza`, `crea`, `busca`, `configura`, `diagnostica`, `planifica`... — as a case-insensitive substring ("Buscar información" matches `busca`) → `Complex`, confidence 1.0.
- No match → `Simple` with confidence 0.0, which means "not resolved, keep going."

**Niveles 2 and 3 — embedding and logistic regression.** I will admit these for what they were: stubs, feature-gated behind an empty `classifier-embedding` flag, always returning an error. Every honest design doc needs a "scheduled for removal" section; pretending a stub is a tier is how architectures rot. They earned their place in the cascade only as placeholders for measurements nobody had taken yet.

**Nivel 4 — SLM fallback.** One HTTP call to a small model with a system prompt that says *"Respond with EXACTLY one token: SIMPLE... or COMPLEX..."*, `max_tokens: 1`, `temperature: 0.0`, and a hard 800 ms timeout (configurable via `CLASSIFIER_FALLBACK_TIMEOUT_MS`). One token in, one token out. The classifier never gets to be the expensive part of the pipeline.

## The deliberate bias: when in doubt, over-equip

If the heuristic passes and the fallback is disabled, errors out, or blows its timeout, the pipeline returns `Complex` with `confidence = 0.0`.

That is a design decision, not a bug. A `confidence 0.0` Complex turn means "I have no idea what you asked for — assume it mattered." Misrouting a real task to a casual low-capability mode loses you the task; misrouting a "gracias" to a tools-enabled turn costs you a few tokens and a slightly drier tone. The errors are not symmetric, so the classifier is not symmetric.

## What the router may change — and what it may never touch

Here is where the post earns its title. Once the intent is decided, the router tunes the request:

| Intent | temperature | tool_choice |
|--------|-------------|-------------|
| Simple | higher | `"none"` |
| Complex | lower | `"auto"` (forced tool use → `"required"`) |

But the `tools` array — the full JSON schema of every available tool — is sent on **both** kinds of turns. Always. The rule pinned in the repo is blunt: *never strip the `tools` array based on intent. Temperature and `tool_choice` may change; the schema must not* (issue #191).

Why? Because the inference server caches the prompt prefix. With automatic prefix caching (the same KV-cache machinery behind the `max-num-seqs` bottleneck), the server reuses computed key/value blocks for any prefix shared with a previous request. Tool definitions are serialized into the system portion of the prompt *before* the conversation. The moment you toggle the tools array between turns — this turn Simple (no tools), next turn Complex (20 tool schemas) — the prefix diverges at the first schema token, every cached block after it is invalidated, and you pay full prefill on every mode switch.

Which, in a voice assistant, is every other turn.

`tool_choice: "none"` is the escape hatch that makes this work: it tells the model "tools exist, do not call them" while leaving the schemas — and the cache prefix — exactly where they were. `tool_choice` is a sampling-side parameter: it does not enter the cached prefix at all. You get the behavioral routing you wanted at the price of a string field, instead of a cache-busting rebuild of your prompt's head.

The counterintuitive lesson: the *classification* can be as clever as you want, but the *request shape* must be invariant. A router that varies the request structure is a router that re-prefills. Let the cheap decision change only the fields the server does not bake into the prefix.

## The geometry of a router you never notice

Look at the cost envelope:

- Heuristic hit (greetings, keyword-bearing commands — most of real traffic): **0 ms**. It is a string comparison that happens while the mic is still settling.
- Fallback path: bounded by the **800 ms** timeout, and it only runs on genuinely ambiguous text — and only if you enable it (`CLASSIFIER_ENABLE_FALLBACK`, off by default).
- Cache penalty: **0 ms**, because the tools array never moves.

A router whose worst case is a hard timeout on a rare branch, and whose cost on the common branch is below the noise floor of your STT, is a router that does not exist from the user's perspective. That is the goal. Latency in a voice loop is not a metric to optimize; it is a metric to hide.

## Why not just "always tools"?

You could skip the classifier entirely and run every turn in Complex mode. Honestly, it would mostly work. Here is what you give up:

1. **Tokens.** ~20 tool schemas ride in every request prefix. Prefill caching absorbs the *compute*, but they still shape the prompt and the model's attention every single time.
2. **Temperature.** Tools-first priming pushes small local models toward clipped, transactional phrasing. "Gracias" answered with the personality of a CLI is a real cost you only hear in a voice product.
3. **Tool-call reliability on small models.** Local models in the 3–8B range hallucinate tool calls when tools are always armed. `tool_choice: "none"` on casual turns removes the temptation; the schemas stay for the cache, the option to misuse them goes away.

The router exists because those three costs are real on *casual* turns and negligible on *task* turns — and a heuristic reading one word tells you which one you are in almost for free.

## Epilogue: the router retired, the rule stayed

The honest ending: intent classification turned out to be so cheap — so dominated by the greeting/keyword heuristic — that the temperature-split machinery earned its retirement (commit `9db0647`, "Remove intent classifier", Aug 2026). The fallback SLM never justified its complexity in production.

But the rule from #191 survived the purge and lives on, enforced by unit tests in the LLM client: the full tool schema ships on every request, with a comment pointing at the issue it came from. It is the part of the experiment worth keeping — because it was never really about intent classification. It was about learning which fields of your request the inference server is allowed to remember, and designing everything else around that.

Classify cheaply, bias toward over-equipping, and never — *never* — strip the tools array.
