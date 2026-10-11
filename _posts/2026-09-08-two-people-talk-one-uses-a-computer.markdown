---
layout:     post
title:      "Two People Talk, One Uses a Computer: Where Voice Beats the Keyboard"
subtitle:   "Voice is not a replacement for typing — it is a parallel channel that unlocks interaction patterns no hands-on-keyboard setup can produce"
date:       2026-09-08 00:00:00
author:     "Daniel Vela"
locale:     en
---

The dominant narrative about voice interfaces is a substitution story: voice replaces typing, speech recognition replaces the keyboard, and the goal is to type less. That framing guarantees voice loses, because for most keystroke work the keyboard is simply better — faster, more precise, silent, and it doesn't announce your medical diagnosis to the open office.

There is a better framing, and it comes out of an analysis I ran over my own voice-assistant project (Seneschal, a Rust pipeline: STT → LLM → TTS → tools). The real value of voice is not that it replaces the hands. It is that it frees them. Voice is a **parallel channel**, and parallel channels unlock patterns that are physically impossible when someone's hands are on the keyboard:

> Person A speaks to Person B — a colleague, a client, a student, a patient.
> Person B speaks to the assistant, which controls the computer.
> The assistant retrieves, processes, acts, and speaks back.
> Person B never breaks eye contact with Person A. Never touches the keyboard.

The second person keeps their attention where it actually matters — on the human in front of them — while the machine is operated as if by a third hand. That is the product. Not "hands-free texting." Sustained eye contact and flow state *while* driving a computer.

## The anchor case: the code reviewer who never looks at the screen

Take the scenario that made the pattern click for me: pair programming and code review.

A senior developer reviews a junior's pull request. The junior explains their approach out loud. The senior, eyes on the junior, says: "open `src/pipeline/mod.rs`, line 42." The file opens. "Run `cargo test fsm`." It runs. "Show me the diff on `feature/x`." The diff appears. The entire review navigates the codebase by voice while the reviewer's attention — and the junior's — stays in the conversation.

What surprised me when I checked the feature list against the actual code was how little new engineering the pattern needs. Every piece already exists in the pipeline:

| Capability needed | What already exists | Work required |
|---|---|---|
| Open a file at a line | `read_file` tool | repurpose |
| Run tests, show diffs | `run_shell` tool (`cargo test`, `git diff`) | repurpose |
| See what the user sees | `take_screenshot` tool | repurpose |
| Map the project | `rg --files` piped through the LLM | prompt design |
| Architecture questions | `deep_research` tool | repurpose |

Every row is LOW complexity because none of it is new machinery — it is existing tools pointed at a new workflow. The genuine innovation is a system prompt ("you are a senior Rust reviewer; prioritize semantics over syntax") and the choreography. I estimated the whole vertical at one to two days of work on top of the existing assistant. The moat, if there is one, is the pattern recognition — not the code.

## Why a cloud chatbot cannot give you this

Once you look at what the pattern actually requires, the local-first argument stops being an ideology and becomes a checklist. Three requirements fall straight out of the scenario:

**Sub-second latency, in both directions.** A conversation between two humans tolerates no awkward pauses. If Person B asks the machine something and gets a two- and a half-second stall plus a spinner, they will — correctly — just type it. The moment they type it, the pattern is dead: their eyes left the room. This is why the pipeline is tuned end to end for time-to-first-audio, not for quality of prose.

**The microphone is always on, at the desktop, in a room with other people.** An always-listening ambient channel is something most people will not route through a third-party server, and should not. The audio never leaves the machine in this design because there is no design that survives the trust conversation otherwise.

**Full permission to operate the operating system.** Shell commands, files, screenshots, apps. A cloud assistant would need root-level delegation over someone's dev box to do what "run the tests" requires here. Handing that to a vendor is a non-starter for most of the people who'd benefit most from it — which is precisely why the tool has to run where the keyboard is.

None of this is a manifesto. It is just what the interaction pattern demands, and the demands happen to all point the same direction: on-device.

## The pattern repeats — that is the interesting part

What started as one scenario turned into a survey of eight niches, and the reason to write this post is that they are all the *same system*. The architecture is identical in every one: always-on STT, local LLM with a domain system prompt, TTS, a tool pool over the local machine, SQLite for memory across sessions. What changes is the prompt and the tool selection. Nothing else.

Three of them make the point better than a full list:

**The field technician.** A junior tech on a wind turbine, in gloves, at height, reads a measurement out loud: "Gearbox oil is 3.2 liters — in spec?" The assistant checks the manual and answers without the tech ever taking a glove off. Here the local-first requirement flips from preference to physics: there is no cell signal nacelle-side. An offline pipeline (whisper.cpp + a local model) is the only architecture that exists for this job.

**The podcast producer.** Mid-interview, the producer murmurs "capture that — the blockchain regulation bit." The moment gets timestamped; show notes and pull-quotes come out after the tape. The producer never looks down from the guest. An existing pipeline quirk — accumulating partial transcripts before end-of-speech is detected — happens to be exactly the primitive real-time tagging needs.

**The game master.** A tabletop GM tracks an entire campaign — NPCs, initiative, lore — without ever glancing at a laptop, because a screen at the table kills the table. A single existing tool for recovering historical context from archived sessions covers most of the vertical. It is the cheapest of all eight to build: prompt engineering and nothing else.

Same bones every time. That is the evidence that "two people talk, one uses a computer" is a *pattern*, not a use case — and it is why the analysis ranked every one of them by what tools already existed before it ranked them by market size. In this architecture, feasibility is almost entirely reuse.

## The actual entry cost

If you want to try the pattern on your own machine, the honest bill of materials, from the project's own numbers:

- **STT, offline:** pluggable — Whisper via whisper-rs (99 languages, Metal/CoreML), NVIDIA Parakeet via ONNX Runtime (25 languages, fully offline), or macOS `SFSpeechRecognizer` running on the Neural Engine (~40 languages). One environment variable to swap them.
- **LLM:** any OpenAI-compatible local server; the pipeline streams and reuses the KV cache for sub-second first-token latency, keeping the whole turn under roughly three seconds end to end — the budget where conversation still feels like conversation.
- **Tools:** file read, shell, screenshot, app launch, web search, MCP — an MCP server or a shell script is a valid integration point for anything domain-specific.
- **Memory:** a SQLite file. Sessions survive restarts; so does the "what did the dragon's name used to be" question three campaigns later.

No cluster, no fine-tuning, no vendor contract. One machine, already next to the two people who are talking.

The keyboard is not going anywhere — for the work that lives in the fingers, it keeps winning. The interesting question was never "can voice replace typing." It was: what happens when the typing hand is freed for the one thing no keyboard was ever able to protect — the look someone gives you while they explain what they meant.
