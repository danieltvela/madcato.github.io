---
layout:     post
title:      "Building a Home GPU Lab: Tesla with RTX PRO 6000 for Local LLM Inference"
subtitle:   "Why VRAM is the only number that matters"
date:       2026-05-30 06:00:00
author:     "Daniel Vela"
locale:     en
lang-ref:   home-gpu-lab
---

> I'm an iOS developer who got curious about local LLMs and ended up building a homelab with two NVIDIA GPUs, three machines, and way too many Docker containers. This is how it came together.

# Why a Dedicated AI Workstation?

It started as curiosity. I wanted to experiment with local models — run them privately, iterate quickly, not depend on cloud APIs. But consumer GPUs felt inadequate for the sizes I cared about. You hit VRAM walls fast when you want to serve a 27-billion-parameter model without collapsing into swap.

So I committed to a proper build: a workstation designed from day one to host inference, image generation, and all the plumbing around it. I named it **Tesla**.

# The Hardware

Tesla is built around the **AMD Ryzen 9 9950X**, 96 GB DDR5, and an **ASUS ROG Crosshair X870E Hero** motherboard with dual PCIe x16 slots. Nothing exotic beyond that — the interesting part is the GPU selection.

| Component | Spec |
|---|---|
| CPU | AMD Ryzen 9 9950X 4.3/5.7 GHz |
| Motherboard | ASUS ROG Crosshair X870E Hero |
| RAM | 96 GB DDR5 6400 MHz (2×48 GB) |
| GPU 1 | NVIDIA RTX PRO 6000 Blackwell — 96 GB VRAM |
| GPU 2 | NVIDIA RTX 3060 — 12 GB VRAM |
| PSU | Corsair HX1500i 1500W 80+ Platinum |
| Case | Antec P101 Silent (Full Tower) |
| Cooler | Noctua NH-D15 G2 |
| Storage | Samsung 990 Pro 2 TB NVMe + 980 1 TB NVMe + FireCuda 530 1 TB + 2× BX500 2 TB SATA |
| OS | Debian 13 |

## Why the RTX PRO 6000?

The RTX PRO 6000 Blackwell gives you **96 GB of VRAM** in a workstation card. That is the number that matters for local LLMs. At FP8, a 27B model fits comfortably with room for context. At FP16, you can still load larger models. There simply isn't a consumer card in this range — the GeForce RTX 5090 tops out at 32 GB.

Yes, it costs more than a consumer flagship. But once you factor in that you avoid monthly cloud bills for heavy usage, the ROI starts to look reasonable. And for development work where you iterate dozens of times a day, not waiting on an API call changes everything.

The secondary RTX 3060 (12 GB) arrived from my gaming rig, **Bolt**. I moved the 7900 XTX to Bolt for gaming and brought the 3060 here — a cheap CUDA-capable card that handles secondary image generation tasks without eating the PRO 6000's memory.

## It Is Not Just Capacity — Bandwidth Matters More

Capacity tells you whether a model fits. Bandwidth tells you whether it runs at a usable speed. This distinction is critical when comparing dedicated GPUs to unified-memory systems like Apple Silicon.

Chips like the **M4 Pro** or **M5 Max** advertise large amounts of unified memory — 48 GB, 64 GB, even 128 GB on Max variants. On paper, that looks like plenty. In practice, the memory bandwidth available to the neural engine is a fraction of what a dedicated GPU offers. An RTX PRO 6000 moves data at **~2 TB/s** over its 680-bit bus. An M4 Pro shares its bandwidth pool across CPU, GPU, and Neural Engine, effectively delivering far less to the inference workload.

The consequence is stark: on Apple Silicon, **only Mixture-of-Experts (MoE) models are practically usable** at scale. MoE models activate only a subset of their parameters per token, so they demand less throughput despite their total size. Dense models — which use all parameters for every token — starve for bandwidth and crawl.

On a dedicated NVIDIA GPU, dense models fly. A 27B dense model at FP8 on the PRO 6000 generates tokens at speeds that feel responsive. The same model on an M4 Pro can be orders of magnitude slower, making it impractical for interactive work.

And it is not just about speed. Dense models tend to carry more knowledge and capability per parameter than their MoE counterparts. They were trained on broader corpora, with richer instruction tuning. Losing access to dense models because of bandwidth constraints means losing access to a significant portion of what modern LLMs can do.

That is why, for serious local inference, a dedicated GPU is not a luxury — it is a requirement.

# The Rest of the Homelab

Tesla doesn't work alone. Here's the full picture:

**Bolt** — Ryzen 5 5600X, 32 GB, RX 7900 XTX. Pure gaming machine running Nobara Linux. Where the AMD GPU belongs.

**Microx** — Ryzen 7 5700G, 64 GB, 2×8 TB HDD mirror. Runs TrueNAS Scale as my NAS with AdGuard Home, Plex, and Transmission.

**Mac Mini M4 Pro** — 14-core, 64 GB. The daily driver for Xcode and iOS development.

**MacBook Air M5** — Portable work and travel.

# What Runs on Tesla

Tesla hosts roughly twenty services, all orchestrated through Docker Compose and exposed behind an nginx reverse proxy with Cloudflare tunnels for external access:

| Service | GPU | Purpose |
|---|---|---|
| vLLM | PRO 6000 | Qwen3.6-27B-FP8, OpenAI-compatible API |
| ComfyUI | PRO 6000 / 3060 | Image generation (Flux, SDXL) |
| Open WebUI | CPU | Chat frontend with RAG, connects to vLLM |
| Gitea + Postgres | CPU | Self-hosted Git hosting |
| SearXNG | CPU | Private metasearch |
| Browserless | CPU | Headless Chrome |
| Docker Registry | CPU | Local container images |
| Firecrawl + deps | CPU | Web scraping (Postgres, Redis, RabbitMQ, Playwright) |

The PRO 6000 handles both vLLM and ComfyUI — they rarely peak at the same time, so 96 GB is enough for both workloads sequentially. The 3060 catches overflow image generation jobs.

# Lessons Learned

**VRAM capacity and bandwidth are both bottlenecks.** Capacity determines what fits, but bandwidth determines what runs at a usable speed. Unified-memory systems like Apple Silicon look impressive on paper — 64 GB, 128 GB — but their shared bandwidth makes dense models painfully slow. Only MoE models remain practical there. A dedicated GPU with a fat memory bus is what you need for serious inference.

**Dual GPUs help more than you think.** Even a modest secondary GPU like the 3060 frees you from fighting over the primary card. Image generation and inference can overlap without killing each other.

**Docker Compose is the glue.** Twenty services on one machine sounds chaotic until you realize they're all declared in a single compose file. Start, stop, update — one command.

**Self-hosting teaches you about dependencies.** You don't just run an LLM server. You need a registry, a reverse proxy, a search backend, headless browsers, queues... The ecosystem grows organically. Having it all local is satisfying precisely because it's messy.

# Next Steps

The motherboard has two PCIe x16 slots. The second one is empty. When the workload demands it, a second RTX PRO 6000 goes in — doubling to 192 GB of VRAM for even larger models or concurrent workloads. Until then, one card is plenty.

The dream is simple: local models that rival cloud quality, running on hardware I control, in my own office. We're getting close.
