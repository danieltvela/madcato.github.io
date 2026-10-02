---
layout:     post
title:      "Kill the Shared Session: Why a Voice Pipeline Needs Typed Channels, Not a Supervisor"
subtitle:   "How 16 AtomicBool flags and one multiplexed string channel became a watch-based FSM with no coordinator on the hot path"
date:       2026-09-08 00:00:00
author:     "Daniel Vela"
locale:     en
---

Seneschal is a real-time voice assistant: a single Rust binary where a microphone, a VAD, Whisper, an LLM streaming hundreds of tokens per second, a sentence splitter, and a TTS engine all have to cooperate within a few hundred milliseconds. When I sat down to audit how those components actually talked to each other, I found two things: a struct that had quietly become a battlefield, and an instinctive "fix" — a central supervisor — that would have made things dramatically worse.

This is the story of killing both.

## The diagnosis: SharedSession

The pipeline ran six concurrent components — audio capture, the main loop, the LLM task, the sentence task, the TTS task, and a background consolidation task — and they communicated through a single struct of convenience: `SharedSession`. Sixteen fields of `Mutex<String>`, `AtomicBool`, `Mutex<VecDeque>`, and `Mutex<Instant>`, shared by every thread. Every access to any of them was a potential race condition, and every "temporary" boolean was a state machine in denial.

Worse was the multiplexing. One pair — a `transliterated_text` string plus a `vad_finish.notify()` signal — carried **five distinct meanings**:

1. Transcribed user speech
2. System notifications injected into the conversation
3. Memory-consolidation results
4. Questions arriving from external ACP agents
5. The initial greeting at startup

When the LLM task woke up on that `Notify`, it read whatever string happened to be in the mutex and could not tell which of the five worlds it was waking into. Text meant for one purpose could be overwritten by another producer before the consumer read it. This is what happens when a channel has no type: semantics leak into convention, and convention leaks into bugs.

Meanwhile, cancellation was one `broadcast` used for two very different intents: "the user just interrupted me" (barge-in, act within milliseconds) and "consolidation needs the LLM to pause politely." Same signal, opposite urgencies, no priority between them.

## The patterns with names and dates

Before redesigning, I looked at what the field already knew.

**Statecharts** (David Harel, 1987) formalized what my boolean flags were fumbling toward: a system's behavior should be declared as an explicit finite state machine, with states, events, and legal transitions — not inferred from a cloud of independent bits. Seneschal already had obvious states (Idle, Listening, Thinking, Speaking); nothing in the code said so.

**The Actor model** (Carl Hewitt, 1973) supplied the alternative to `SharedSession`: components with *private* state that communicate only by passing messages. Actors never share mutable state — which is precisely the property `SharedSession` violated.

**Pipecat**, a production voice-agent framework, draws the sharpest practical line: pipelines carry two kinds of frames. *Data frames* (audio, text, tokens) flow forward; *control frames* (start, cancel, interrupt) can flow in any direction. My system mixed both in the same channel — the transcript string carried user data *and* system control notifications.

**ARINC 653**, the avionics standard for time- and space-partitioned systems, forced me to write down what each stage actually needs:

| Priority | Component | Deadline |
|----------|-----------|----------|
| CRITICAL | VAD detection + barge-in | < 50 ms |
| HIGH | STT transcription | 200–500 ms |
| MEDIUM | LLM inference + TTS | 500–1000 ms |
| LOW | Consolidation, memory | no deadline |

High-priority work must be able to interrupt low-priority work, never the reverse. One broadcast signal serving both barge-in and a polite consolidation pause violates that rule by construction.

## The wrong fix: a supervisor

The textbook reflex against shared mutable state is a coordinator: one task that owns all state, receives every event, and re-emits commands to the workers. A tidy `PipelineSupervisor`. A serialization point with a nice name.

I rejected it, and the reason is a number: the LLM streams hundreds of tokens per second. A supervisor sitting between the LLM task and the sentence task would be a single-threaded funnel processing every `LLMToken` frame, every sentence boundary, every interruption — adding a hop, a queue, and a contention point precisely in the path that feeds the "first word in under a second" budget. Coordinators don't remove races; they just make you wait in line.

## The right shape: typed channels plus a watch-based FSM

The redesign kept the actors — the task decomposition was already correct — and replaced only *how* they communicate. Two planes:

**Control plane.** One `tokio::sync::watch<PipelineState>` channel holds the global state as an enum:

```rust
pub enum PipelineState {
    Idle,
    Listening { utterance_id: u64 },
    Thinking  { utterance_id: u64 },
    Speaking  { utterance_id: u64 },
    Paused    { reason: PauseReason },
}
```

A `watch` channel keeps only the *latest* value: one writer at a time, any number of readers, no queue, no backlog. **Each actor that owns a transition writes it directly** — the VAD actor moves the state to `Listening`, the LLM actor to `Thinking`, the TTS actor to `Speaking`. No coordinator sits in between. The sixteen booleans collapse into one enum variant: `llm_busy` *is* `Thinking`, `consolidation_active` *is* `Paused { Consolidation }`. Illegal combinations like `llm_busy && !listening` stop being representable.

Cancellation splits into two broadcasts with different writers and different audiences: `barge_in_tx` (only the VAD sends it; every actor cancels immediately) and `pause_tx` (only consolidation sends it; only the LLM listens). Priority bands, made structural.

**Data plane.** Every flow gets its own typed channel:

```
VAD/STT  --mpsc<TranscriptReady>-->  LLM actor
LLM      --mpsc<LLMToken>-------->  Sentence actor
Sentence --mpsc<SentenceReady>--->  TTS actor
```

Every frame carries an `utterance_id`, so one user turn can be traced end to end through logs and latency metrics. The five meanings that used to share one string became distinct variants — `TranscriptReady`, `SystemNotification`, `AgentResult`, `TextInput` — each with its own producer and its own semantics. Audio chunks stay on their own bounded channel, deliberately outside the frame enum: too high-volume to route.

A supervisor still exists — as a pure *observer*. It subscribes to the `watch` receiver and logs every transition for the TUI and diagnostics. It never sits between actors, because it never needs to: the state is already in one place, and everyone already reads it.

The migration was mechanical enough to tabulate — every `SharedSession` field mapped to a typed replacement (`transliterated_text` → `mpsc<TranscriptReady>`, `assistant_text` → `mpsc<LLMToken>`, `sentences` → `mpsc<SentenceReady>`, `llm_busy` → `PipelineState::Thinking`, and so on) until the struct stood empty and could be deleted.

## Refactor, not rewrite

A full rewrite onto a real actor framework — supervision trees, SCXML statecharts, priority scheduling — was estimated at two to three weeks. The incremental refactor took three to five days: define the frame enum, add the FSM and the watch channel, migrate fields one by one (build and test after each), split the cancellation signals, delete `SharedSession`.

The rewrite was not wrong in general — it is appropriate for a product with multiple engineers and contractual latency SLAs. It is overhead for a single-user bot on one machine whose existing task structure was already the right one. The lesson is the boring one: diagnose whether your problem is structure or mechanism. Mine was mechanism, and mechanism refactors cheaply.

## Frozen as law

The final shape is now written into the project's `CONSTITUTION.md` under the non-negotiable architecture section: pipeline state is an explicit FSM carried by a single `watch<PipelineState>` channel, each stage is a tokio task with private state that talks only through typed channels, and — the clause that started this whole detour — *no central coordinator sits on the hot path*. Amending either requires explicit deliberation.

Writing the shape of the system down, after killing the shared session, is what keeps the next contributor (or the next agent) from helpfully reintroducing the supervisor.

Two rules, earned the slow way:

1. **Shared mutable state is not a performance problem, it is a semantics problem.** `Mutex<String>` + `Notify` was not "slow" — it was ambiguous. Types are the fix; locks are a placebo.
2. **A coordinator is a bottleneck wearing a diagram.** If the global state is one small latest-value enum, a `watch` channel gives everyone a view of it for free — and the coordinator's inbox is exactly the hot path you cannot afford.
