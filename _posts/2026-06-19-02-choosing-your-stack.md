---
layout: post
title: "Choosing Your Stack: Subscriptions, APIs & Local Models"
date: 2026-04-11
series: "Start Your AI Homelab"
part: 2
description: "The three ways to get AI into your homelab, honest trade-offs, the model landscape, and what I'd pick at each budget."
---

Last post we covered what large language models actually are and why they're worth running thoughtfully rather than just firing prompts into a chat box. Now we hit the first real decision: *how* do you get AI into your setup? There are three fundamentally different approaches, and which one you reach for first depends on your budget, privacy tolerance, and hardware situation.

Let's go through them properly.

---

## The Three Ways to Get AI

### 1. Chat Subscriptions

The simplest entry point. You sign up for a monthly plan — something like Claude Pro, ChatGPT Plus, or Gemini Advanced — and you get a polished web interface, often a mobile app, and access to the provider's best models. Some plans at the higher tiers also bundle coding-agent tools that can run on your machine and interact with files, terminals, and browsers.

The appeal is zero setup. You're up and running in five minutes. You're also sharing your data with the provider unless you've gone through enterprise agreements, and you're rate-limited — peak hours can feel sluggish, and hitting your usage cap mid-afternoon is annoying when you're in flow.

For homelab purposes, subscriptions are great for *your own* daily driving — the interactive, conversational stuff, writing assistance, research, ad-hoc code review. They're less suited to automated pipelines where you need to fire thousands of requests unattended.

### 2. Pay-Per-Token API

This is where homelabs get interesting. Every major provider offers an API where you authenticate with a key, send a request, get a response, and pay for the tokens you used. No monthly flat fee — or a small base fee — and you only pay for what you actually consume.

This is the right tool for automation. Want a nightly job that summarises your RSS feeds? A webhook that summarizes inbound emails before they hit your inbox? A script that reviews your code before you commit? API calls. You write the code, you own the pipeline, and the cost scales with usage rather than being a flat overhead whether you use it or not.

The trade-off is you need to write code (or use a tool/framework that handles it for you), you need to manage API keys responsibly, and costs can surprise you if you're not watching. Metering and budget alerts are your friends — every provider offers some version of spend limits.

Privacy note: your prompts and data still travel to the provider's servers. For sensitive local data, that's a consideration.

### 3. Self-Hosted Local Models

The third path: download open model weights and run them on your own hardware using a runtime like **Ollama** or **LM Studio**. No API key, no monthly bill, no data leaving your machine. You spin up a local server, point your tools at `localhost`, and you're off.

The catch is capability and hardware. Open models have improved dramatically, but the frontier models from the major labs are not open-source — what you can run locally is a step or two behind the cutting edge. And "running locally" has real hardware constraints that we'll cover below.

Still, for private data, offline tasks, high-volume cheap inference, or just the satisfaction of owning your whole stack, local models are genuinely useful — not just a novelty.

---

## Trade-Off Table

| Approach | Cost model | Privacy / data control | Capability ceiling | Setup effort | Best for |
|---|---|---|---|---|---|
| Chat subscription | Flat monthly (~$20/mo) | Data goes to provider; some enterprise options | Frontier models; best for current gen | Minutes | Daily interactive use, coding agents, exploration |
| API (pay-per-token) | Pay what you use; no/low base | Data goes to provider; enterprise options exist | Frontier models same as above | Low–medium (need to write integration code) | Automation, pipelines, webhooks, custom agents |
| Self-hosted local | Hardware upfront; near-zero marginal | Fully private; never leaves your machine | Behind frontier; scales with your hardware | Medium–high (download, configure, manage) | Private data, offline use, cheap high-volume inference |

---

## The Model Landscape

Here's a fair read of what's out there. I'll be honest about where I lean, but I'll try not to make this a commercial.

### Claude (Anthropic)

Anthropic's Claude family — currently the Claude 4 generation, with Opus 4.8 as the heavyweight and Sonnet 4.6 as the faster, cheaper workhorse — is available via subscription (Claude Pro) and via the Anthropic API. The subscription tier includes Claude Code, a terminal-native coding agent that I use as my daily driver.

Where Claude stands out, in my experience: complex reasoning tasks, long-context work (reading a whole codebase, not just a file), and agentic tasks where the model needs to plan, use tools, and recover from errors without going off the rails. It also tends to be more careful about saying "I don't know" rather than confidently hallucinating, which matters when you're building things you'll actually use.

I lean on Claude because it's where I get the most reliable results on the kinds of things I do — coding, analysis, system design. That's a personal data point, not a universal truth.

### GPT (OpenAI)

OpenAI's GPT lineup — its current flagship generation plus the reasoning-focused variants — is available via ChatGPT Plus subscription and the OpenAI API. It's the model most people encountered first, has the broadest third-party integration story (more tools and frameworks support it out of the box than anything else), and the API is very well documented.

For general-purpose use, for integrations where you want plug-and-play framework support, or for teams already familiar with the OpenAI interface, it's a solid choice. The new reasoning-oriented variants are worth watching for technical tasks.

### Gemini (Google)

Google's Gemini family — available via the Google AI Pro subscription (the tier formerly branded Gemini Advanced) and the Google AI / Vertex AI APIs — has a genuinely impressive multimodal story. If you're working with images, audio, video, or mixed-media inputs, Gemini's native handling of those modalities is strong.

It also has very long context windows in some tiers — useful if you're feeding it large documents (the current frontier models from all three providers now reach into the million-token range). Google's enterprise integration (Workspace, Drive) can be a selling point if that's your world.

### Local / Open Models (Llama, Qwen, Mistral, and friends)

This is the most active category in the current period. Meta's Llama family, Alibaba's Qwen models, and Mistral's lineup are all available as open weights — meaning you can download them and run them yourself.

The quality gap between open and frontier has narrowed significantly. For well-scoped tasks — summarization, classification, code generation for common languages, Q&A over your own documents — a capable open model running locally can do genuinely useful work. For open-ended reasoning, complex multi-step tasks, or anything requiring broad world knowledge, frontier models still have an edge.

The tool of choice for running these is **Ollama**: one command to pull a model, one command to run a server, and it exposes an API that's broadly compatible with OpenAI's format, so your existing code often just needs a URL change.

---

## Hardware Reality for Local Models

This is where beginners often get tripped up. Here's a practical breakdown:

**Small models (1–7 billion parameters):** These run on a normal laptop with no GPU required. You can run a solid 7B model on 8–16GB of system RAM and get useful responses, though it'll be slower than a cloud API. Good for experimentation and low-throughput tasks.

**Mid-size models (roughly 13–34 billion parameters):** You want 16–32GB of system RAM for CPU-only inference, or a GPU with 8–12GB of VRAM for reasonable speed. A gaming GPU in this range is a common homelab choice. Inference gets fast enough to feel interactive.

**Large models (70B and up):** You're now in the territory of serious GPU hardware — multiple high-VRAM GPUs or specialized accelerator cards. This is not the beginner tier. The costs are meaningful and the power draw is real.

A practical starting point: if you have a machine with 16GB RAM, pull a 7–8B model via Ollama and see how it performs on your actual tasks before committing to hardware. Many people find that small local models handle their offline/private needs just fine.

---

<div class="sidebar-note" markdown="1">
**How I run mine.** I use a three-layer approach. My subscription handles interactive daily driving — conversations, ad-hoc code review, anything where I want the strongest model available with no friction. The API powers my automated jobs: nightly summaries, inbound classification, custom agents that run on a schedule. And a local model runs on my homelab server, accessible only from inside my network, for tasks where I don't want data leaving the building — indexing personal notes, processing local files, that kind of thing. I don't use the local model for everything; I use it where privacy matters or where the volume would make API costs add up. The three tiers coexist comfortably.
</div>

---

<div class="callout" markdown="1">
**What I'd pick at each budget.**

**$0 — Local only.** Pull Ollama, grab a small 7–8B model (a current Llama or Qwen is a safe bet), and start experimenting. You'll learn a lot about prompt engineering and model behavior with zero spend. Capability ceiling is real, but so is the learning value.

**~$20/month — One good subscription.** Pick one frontier provider and go all-in. I'd pick Claude Pro for the coding agent and the quality on complex tasks, but if you're already in the OpenAI or Google ecosystem, the equivalent tiers are genuinely good. Don't split across three subscriptions at this budget — depth beats breadth.

**~$20/month subscription + metered API — The homelab sweet spot.** Subscription for your personal daily use; API for automation. Set a monthly API budget cap (start small — $10–20/month goes further than you'd expect for personal pipelines), write your first automation, and grow from there. This is the tier where your homelab starts doing things for you rather than just answering questions.
</div>

---

## Making the Call

There's no wrong answer here, only trade-offs that fit your situation differently. My honest recommendation for a beginner: start with a subscription, use it for a month, identify one repetitive task you do manually that AI could help with, then try building that as an API call. That transition — from user to builder — is when a homelab starts feeling like a homelab.

If privacy is your primary concern from day one, start with Ollama on whatever hardware you have and work up from there.

---

**Next up:** [Getting the VM ready — setting up a clean Linux environment for your AI homelab]({% post_url 2026-06-26-03-prepare-the-vm %})

**Previously:** [AI Foundations — what LLMs actually are and why they're worth running thoughtfully]({% post_url 2026-06-12-01-ai-foundations %})
