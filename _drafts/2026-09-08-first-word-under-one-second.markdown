---
layout:     post
title:      "The First Word in Under One Second: Streaming TTS and the Physics of Interruption"
subtitle:   "Latency in voice is not a benchmark number — it is a chain of streaming decisions, and a cancellation problem you have to solve before you sleep"
date:       2026-09-08 00:00:00
author:     "Daniel Vela"
locale:     en
---

Every voice assistant demo ends with someone saying "it's so fast." Nobody can say *why*. Latency in a voice pipeline is not a number you measure at the end — it is the sum of every streaming decision you made (or refused to make) along the chain: microphone to VAD, VAD to STT, STT to LLM, LLM to TTS, TTS to speaker. If any single stage buffers "until it is complete," the whole budget is gone.

Seneschal, my self-hosted voice assistant, declares its target in the project constitution itself: **the first word heard by the user in under one second.** Not time-to-first-token — time-to-first-*word*, the one you actually hear. Here is how the pipeline earns it, and the part nobody puts in demos: making the whole machine stop, instantly and idempotently, the moment you talk over it.

## Never wait for the whole answer

The naive pipeline is a relay race: the LLM finishes the full response, then the TTS gets the text, then audio plays. A ten-sentence answer is ten seconds of silence before the first syllable.

The fix is to treat the response as a stream all the way to the speaker. The LLM emits tokens over streaming SSE; they flow into a `SentenceSplitter` that buffers only until it hits a punctuation boundary, then emits a `SentenceReady` message. The TTS synthesizes and plays sentence N while the LLM is *still generating* sentence N+1:

```
LLM client (streaming SSE)
  → mpsc<LLMToken>
  → SentenceSplitter (buffer until punctuation boundary)
  → mpsc<SentenceReady>
  → TTS (Kokoro ONNX / AVSpeechSynthesizer)
  → AudioOutput (CPAL)
```

This is written into the architecture as a contract, not an optimization: the constitution states the response "is not buffered entirely before TTS," and gives the rationale in one line — *first word heard by the user in under 1 second.* Once that invariant exists in writing, a future refactor that batches "for simplicity" is a constitution violation, not a code style opinion.

The budget is real, though. The first word only lands under a second if nothing upstream of the splitter ate the time first.

## The latency chain is a chain of dials

Before the model says anything, the machine has to decide you *finished saying* something. That is the most expensive decision in the pipeline, and it is a tunable trade-off, not a bug:

| Stage | Cost | Dial |
|-------|------|------|
| VAD end-of-speech | `VAD_SILENCE_MS`, default **300 ms** of silence before the utterance fires | drop to 150 ms for snappier turns |
| Audio chunking | `AUDIO_CHUNK_MS` = **100 ms** per processing chunk | fixed cadence across capture and playback |
| STT (macOS `SFSpeechRecognizer` on the Neural Engine) | **180–400 ms** end-of-utterance | provider choice |

Two of these dials deserve honesty.

**`VAD_SILENCE_MS` is a bet on your pauses.** At 300 ms the assistant waits a third of a second after you stop talking — long enough to survive a mid-sentence "um." At 150 ms it feels telepathic and occasionally interrupts you mid-thought. The mitigation that makes the aggressive setting survivable: the speech buffer *accumulates across pauses*, so if the pipeline fires early and you resume speaking, no audio is lost — the next utterance simply carries what came before. You buy responsiveness with a possible early trigger, not with dropped words.

**Provider choice moves ~200 ms at a stroke.** Batch providers (Whisper, Parakeet via ONNX) buffer audio during speech and transcribe on silence — the transcription cost lands entirely *after* you stop talking. Apple's `SFSpeechRecognizer` runs incrementally on the ANE and delivers end-of-utterance results in the 180–400 ms band. Same Silero VAD in front, wildly different wall-clock behind it. On constrained hardware, the STT provider is a latency decision disguised as a model decision.

Add LLM time-to-first-token and TTS synthesis time for one short sentence, and you can see why the under-one-second target is only reachable when *every* stage streams. One non-streaming stage silently converts the whole pipeline into the relay race you thought you left behind.

## Barge-in is distributed cancellation wearing a voice costume

Interruption looks like a feature ("if the user speaks, stop talking"). It is actually a consistency problem: at the moment of interruption, work is in flight in four places at once — an open HTTP stream to the LLM, a partially filled splitter buffer, a synthesis queue, and a speaker stream mid-chunk. Stopping "the assistant" means cancelling all of them, from one signal, in the right spirit.

The constitution defines the cascade:

1. VAD detects `SpeechStart` and sends cancellation on a `broadcast barge_in_tx`.
2. The LLM task aborts the HTTP stream.
3. The sentence task drains its buffered tokens.
4. The TTS task kills playback via a `play_cancel: AtomicBool`.

Two properties make this actually work:

**Cancellation is per-chunk, not per-sentence.** Playback uses a blocking `play_blocking(samples, sample_rate, cancel)` loop: audio is split into `AUDIO_CHUNK_MS`-sized chunks (100 ms), and *before each chunk* it checks `cancel.load(Ordering::Relaxed)` and returns immediately if set. Worst-case audible latency after an interruption is therefore one chunk — a tenth of a second — instead of "whenever this sentence finishes." An `AtomicBool` polled per chunk beats any clever async teardown, because it cannot be forgotten by a future code path.

**Cancellation is idempotent.** Every actor must tolerate receiving the cancel signal *even after it has already completed its work.* Broadcast channels and in-flight streams make signal ordering unpredictable — the LLM task may finish and close its stream a microsecond before the cancellation arrives, and that must be a no-op, not a panic or a stuck state. "Immediate *and* idempotent" sounds like belt-and-suspenders paranoia until you have heard a voice assistant that lost one cancellation and talked over its own user forever.

## Two signals, or your memory cleanup interrupts you

One more mistake worth describing, because it is the kind you only make once. The original design had a single cancellation broadcast reused for two very different events:

- **The user is speaking** — every actor must drop everything, *now*.
- **Context consolidation is running** — the LLM should pause politely while memories are summarized; nothing else should stop.

Sharing one channel meant the consolidation pause rippled into actors that had no business reacting to it, interfering with barge-in behavior at exactly the wrong moments. The rewrite split the control plane into two dedicated broadcasts with different senders and audiences:

```rust
let (barge_in_tx, _) = broadcast::channel::<u64>(4); // payload = utterance_id; VAD → all
let (pause_tx, _)    = broadcast::channel::<()>(2);  // consolidation → LLM only
```

`barge_in_tx` carries an utterance id, so actors can tell *which* turn they are killing. `pause_tx` is listened to by exactly one actor. A signal's meaning is its sender and its audience — one channel, two meanings, is a race condition with good documentation.

## Say you're thinking: the filler tone

There is a hole in the streaming design worth naming: while a tool call runs, no audio exists to stream. The user asked something, the machine went quiet, and silence reads as death.

The answer is deliberately dumb. When a tool call is dispatched, a `FillerController` starts a background cue: a **440 Hz sine at amplitude 0.05**, in bursts of **200 ms on / 800 ms off**, with **10 ms fade-in/out** to avoid clicks. It stops the moment the tool returns or a barge-in arrives. The sound says "I'm working" in the one language — periodicity — that costs nothing and never competes with your voice.

Even the filler obeys the cancellation rules. It runs on a dedicated blocking thread (not the async runtime, so it cannot be starved by the work it is announcing), and cancellation uses a **generation counter**: `start()` increments it, `stop()` increments it, and the loop compares the counter every 50 ms tick and exits on mismatch. No joins, no flags with ambiguous ownership — a stale loop notices it is stale on its own.

## What the numbers actually say

None of these numbers — 300 ms, 100 ms, 180–400 ms, one second — came from a benchmark suite. They are the arithmetic of a chain: how much you are willing to wait for a pause, how big your work slices are, and how fast your provider answers. The engineering content of voice latency is not measuring the total, it is refusing to let any stage hold the bag:

1. Stream every stage, or the invariant "speak sentence N while generating N+1" dies quietly.
2. Cancel per-chunk, not per-task — the user's ear is a 100 ms clock.
3. Make cancellation idempotent; ordering will betray you.
4. One signal, one meaning, one audience.
5. Fill silence deliberately, and let the filler obey the same rules as everything else.

Under one second is not impressive physics. It is five unglamorous decisions, each defended in writing, compounding.
