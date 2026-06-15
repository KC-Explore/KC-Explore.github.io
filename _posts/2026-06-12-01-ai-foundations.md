---
layout: post
title: "AI Foundations Without the Hype"
date: 2026-04-04
series: "Start Your AI Homelab"
part: 1
description: "Models, tokens, context windows, and agents — the vocabulary that actually matters, in plain English."
---

Before you plug anything in, it helps to speak the language. Not the academic version — the working version. The handful of terms that keep coming up when you're actually building things and need to know why something behaves the way it does.

This post covers seven concepts. None require a math background. Each one has a direct consequence for what you'll build.

*&larr; Previous: [Start Here]({% post_url 2026-06-05-00-start-here %})*

---

## What a model is

A model is a file. A very large file — often several gigabytes — full of numbers called weights. These weights were produced by a training process where the model read enormous amounts of text, over and over, adjusting itself until it got good at predicting what comes next in a sentence.

The key word is *was*. Training is past tense. Once training ends, the weights freeze. The model stops learning. Its knowledge has a cutoff date, and anything that happened after that date simply doesn't exist to it. It isn't browsing the internet when you talk to it. It isn't consulting a database. It's retrieving patterns baked into those frozen weights.

Think of it like a very well-read person who has been in a sensory-deprivation tank since a specific date. They know an enormous amount about everything up to that point. They can reason, summarize, explain, and generate. They just can't tell you last Tuesday's headlines.

**Why it matters for your homelab:** When you ask a model something time-sensitive and it confidently gives you a stale answer, this is why. You'll learn to feed it current context rather than trusting its training data alone.

The other thing introduced here: models can be *hosted* (running on someone else's hardware, you access via API or subscription) or *local* (running on your own machine). The full trade-off lives in Post 02. For now, just know both exist.

---

## Tokens

Pricing, limits, and speed are all measured in tokens, so it's worth understanding what they actually are.

A token is roughly three-quarters of a word. Not a word — a chunk. Common words like "the" or "is" are single tokens. Less common or longer words get split into multiple pieces. "Homelab" might be one token; "unconstitutional" might be three.

A decent rule of thumb: one page of plain text is roughly 500–700 tokens. A short novel is around 100,000 tokens. A long technical document you want to paste in might surprise you.

**Why it matters for your homelab:**

- Hosted models charge per token (input tokens + output tokens, usually priced separately)
- Every model has a token *limit* — a cap on how much it can handle at once
- Speed is partly a function of token count: more tokens in = more computation

When you're building tools that process documents, summarize logs, or answer questions about long files, token count becomes a real engineering constraint, not an abstract concern.

---

## Context window

The context window is the model's desk.

Imagine you're working at a physical desk. You can spread out papers, reference documents, sticky notes, your own notes from earlier in the conversation. But the desk only has so much surface area. When it's full, something has to fall off the edge — and once it's off the desk, it's gone. The model can't see it anymore.

Everything counts against that desk space: the instructions you gave at the start, the conversation so far, any documents you pasted in, and the model's own reply as it's being written. It's all sitting on the same desk.

Context windows are measured in tokens. A 128,000-token context window sounds huge until you paste in three long documents and ten rounds of back-and-forth conversation. Then you feel it.

**Why it matters for your homelab:**

- Big files can blow the window; you may need to chunk or summarize before passing them in
- If a conversation gets very long, the model starts "forgetting" earlier messages — not because it's ignoring you, but because they've scrolled off the desk
- Larger context windows generally cost more per call (more computation per token processed)
- When building agents that process logs, emails, or long documents, you'll design around window limits constantly

<div class="callout" markdown="1">
**The one thing to remember.** The model only knows what's on the desk right now. Its training gives it general knowledge; the context window gives it *situational* knowledge. Your job as a builder is to put the right things on the desk.
</div>

---

## Prompts vs system prompts

A *prompt* is what you type. It's the question, the instruction, the task. "Summarize this log file." "Write a reply to this email." "What's wrong with this config?"

A *system prompt* is different — it's a standing instruction that shapes how the model behaves, set before the conversation starts. Think of it as a job description handed to the model before it meets you. "You are a terse assistant who always responds in bullet points." "You are a home automation helper; never suggest anything that requires cloud connectivity." "You are a code reviewer; flag security issues first."

The user never writes the system prompt (unless they're building the tool). It runs silently in the background, setting the frame for everything that follows.

**Why it matters for your homelab:** When you build your own tools — a local assistant, an automation agent, a chat interface for your infrastructure — the system prompt is where you inject personality, rules, and constraints. A well-written system prompt is the difference between a generic chatbot and a tool that actually fits your workflow. It also counts against your context window, so keeping it tight matters.

---

## Agents

A chatbot answers questions. An agent *does things*.

The leap is this: give a model access to tools — the ability to read files, run terminal commands, call APIs, search the web — and then let it loop. Ask it to accomplish a goal, and it will figure out what steps are needed, use its tools to take those steps, look at the results, and keep going until it's done or stuck.

This is not science fiction; it's how most modern AI assistants actually work under the hood. The model acts as the brain. The tools are the hands. The loop is the work cycle.

A useful analogy: a very competent junior assistant. You hand them a task — "audit the dependencies in this project and flag anything with a known vulnerability." They don't know everything, but they know how to look things up, they can run a command and read the output, and they'll come back to you with a draft answer. Sometimes they need guidance. Sometimes they get it wrong. But they can do real work, not just chat.

**Model Context Protocol (MCP)** is the emerging standard for how tools get plugged into agents — a consistent interface so that a file reader, a web fetcher, and a database connector all speak the same language to the model. If you see "MCP server" or "MCP tools" in documentation, that's what it refers to. You don't need to understand the spec right now; just know it's the plumbing that connects models to capabilities.

**Why it matters for your homelab:**

- Agents are what make AI actually useful for infrastructure work — not just answering questions, but taking actions
- You can build agents that monitor services, respond to alerts, summarize logs, or draft configurations
- The tool set determines what the agent can touch; designing good boundaries is part of building good agents
- Agents can fail, loop, or get confused — they're powerful and they need guardrails

---

## Hosted vs local

One paragraph, because the full breakdown is Post 02.

*Hosted* means the model runs on the provider's servers and you access it via an API key or subscription. Fast, capable, maintained by someone else — but your data leaves your network, there's a per-token cost, and you depend on their availability. *Local* means the model runs on your hardware: a laptop, a home server, whatever you have. Your data stays home, no per-token cost, no subscription — but you're limited by your hardware, and local models are generally weaker than the frontier hosted ones for the same task. The interesting part of homelabbing is figuring out which tasks belong where.

---

<div class="sidebar-note" markdown="1">
**How I think about it.** I run a mix at home — some tasks go to a hosted model via API because I need the reasoning quality; others run entirely locally because the data is sensitive or the task is simple enough that a lighter model handles it fine. I think of the context window like a grocery bag: it has a fixed capacity, and I get deliberate about what I put in. Stuffing in everything "just in case" wastes space and money. Putting in only the relevant snippet — the specific log section, the one config file — keeps things fast and cheap. Agents I think of as junior staff: capable, worth delegating to, but I wouldn't let one touch anything irreversible without a confirmation step.
</div>

---

## The vocabulary in a sentence

Here's the whole picture in one run:

A **model** is a frozen file of knowledge. You talk to it by writing a **prompt**; a hidden **system prompt** shapes its behavior. Everything it can see at once — your messages, its replies, any documents — fits inside the **context window**, measured in **tokens** (which is also how you're charged). When the model gets **tools** and a loop to take actions on your behalf, that's an **agent**. It can be **hosted** on someone else's servers or running **locally** on your hardware.

That's the foundation. Everything else is details.

---

## Next up

Post 02 goes into the actual decision: what do you run hosted, what do you run locally, and how do you decide? Hardware requirements, model sizes, API costs, and the privacy calculation.

*&rarr; Next: [Choosing your stack: subscriptions, APIs & local models]({% post_url 2026-06-19-02-choosing-your-stack %})*
