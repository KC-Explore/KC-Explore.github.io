---
layout: post
title: "Starter Projects Worth Your First Weekend"
date: 2026-07-10
series: "Start Your AI Homelab"
part: 5
description: "Five AI-homelab projects graded from easy to ambitious — what each one teaches you and why it's a good rung to climb."
---

You have a VM. You locked it down. You have Docker running and a basic reverse proxy you half-understand. This is the point where most tutorials say "congratulations, now go build stuff" and leave you staring at the ceiling.

This post is the stuff.

I've picked five projects in rough order of difficulty. None requires a CS degree. All of them teach you something that compounds — each rung makes the next one easier. Do them in order if you want a curriculum, or jump to whichever itch you need to scratch first.

---

### Project 1 — Run a local model with Ollama and a chat UI

**What it is:** Ollama is a tool that downloads and runs open-weight language models locally — Llama, Mistral, Gemma, and a dozen others — and exposes them over a simple API. Pair it with Open WebUI (a browser-based chat interface) and you have a ChatGPT-alike running entirely on your own hardware, sending nothing to anyone.

**What you'll learn:** This is post 01's theory made tangible. You'll feel the difference between a 7B and a 13B model when your machine slows to a crawl loading the bigger one. You'll hit a context window limit mid-conversation and understand why that number matters. You'll discover that a quantized 4-bit model runs fine on 8 GB of RAM and sounds surprisingly good. None of this sticks from reading — it sticks from watching a progress bar crawl and a model finally answering you.

**Rough effort:** An evening. Ollama installs in one command. Open WebUI ships as a Docker image. If your reverse proxy is already working from earlier posts, you'll have it behind a subdomain in under two hours. The only wildcard is download time — some models are several gigabytes.

**Why it's a good rung:** It closes the loop on everything abstract from the first post. And once the API is running locally, every other project on this list can use it as a backend — you're building infrastructure, not just a toy.

**One thing to watch:** Start with a small model (3B or 7B). It's tempting to grab the biggest model that fits on your machine, but smaller models answer faster and the quality difference matters less than you expect for simple tasks. Get comfortable with the workflow first.

---

### Project 2 — Self-host a useful service you'd actually use daily

**What it is:** Pick a service that replaces something you already use — a bookmarking app, a read-later list, a personal homepage that shows you your calendar, the weather, and quick links. Linkding is a clean, minimal bookmark manager. A homepage dashboard (several options exist in the self-hosting community) lets you wire in status widgets for your other services.

The specific tool matters less than the pattern: you're running a container, pointing your reverse proxy at it, and ending up with a real URL you visit on purpose.

**What you'll learn:** Docker Compose in practice — environment variables, volume mounts, the difference between a container restarting itself and a container that stays dead. Basic reverse proxy routing: how a request for `bookmarks.yourdomain.local` becomes a request to `localhost:8765` on your server. And crucially, you'll learn what it feels like to run something that matters to you, which changes how carefully you treat it.

**Rough effort:** Easy to medium, depending on the service. Most popular self-hosted tools have a `docker-compose.yml` in their README. Budget an evening for setup, and another hour later when you inevitably forget what you did and have to read your own notes (you did take notes, right — more on that below).

**Why it's a good rung:** This one isn't AI. That's the point. Homelab is broader than AI, and the Docker + reverse proxy muscle memory you build here is the exact same muscle you use for every AI service later. It's also a low-stakes environment to break things — if your bookmark manager goes down for a day, the consequences are mild.

---

### Project 3 — A "chat with your docs" setup (RAG)

**What it is:** Retrieval-Augmented Generation, or RAG, is the pattern where you load your own documents — notes, PDFs, articles — into a vector database, and the AI searches that database before answering. Instead of the model hallucinating from its training data, it reads your actual content first.

Tools like Anything LLM or Open WebUI's built-in document mode let you drop in a folder of PDFs and start asking questions within an hour, no code required. If you want to understand what's happening under the hood, there are Python-based tutorials that walk you through building the retrieval layer yourself.

**What you'll learn:** Why embeddings exist — your documents become numerical vectors, and "similarity search" means finding vectors near yours. Why context windows matter in practice: you can't stuff 500 pages into a single prompt, so the retrieval step selects the relevant chunks. What chunking strategy means and why it affects answer quality. This is where the theory of post 01 stops being abstract.

**Rough effort:** A weekend, realistically. The tooling is better than it was two years ago, but you'll spend time on document formatting, chunk size tuning, and being surprised when the model ignores a document that you're certain contains the answer. Budget time to experiment.

**Why it's a good rung:** RAG is the most common pattern in real enterprise AI systems. Understanding it from first principles — not just clicking "upload PDF" in a SaaS tool — is a durable skill. And the moment you get a good answer from your own notes, it clicks why people care about this.

---

### Project 4 — A simple agent that does one useful job on a schedule

**What it is:** An "agent" in the practical sense: a script that uses a model (local or API-based) plus one or two tools to do a task automatically. The canonical beginner version is a morning briefing — fetch your RSS feeds, summarize the ten most recent items, send yourself a Telegram or email with the digest. Another version: watch a folder for new files and summarize or tag them.

You can build this in Python with a cron job and 50-100 lines of code. If you're not a Python person, n8n (a self-hosted workflow automation tool) lets you wire this together visually.

**What you'll learn:** Tool use — what it means for an AI to "call a function" rather than just generate text. The mechanics of a cron job and why scheduled automation is surprisingly hard to get right (time zones, error handling, the job that silently fails for three weeks). Prompt engineering for a specific task: how you instruct the model to summarize vs. extract vs. classify, and why the instructions matter a lot.

**Rough effort:** Medium to ambitious. The happy path takes an afternoon. Getting it robust — so it handles an RSS feed that goes down, or a model response that's malformed, or a cron job that fires twice — takes longer. Start simple and add resilience later.

**Why it's a good rung:** This is where "I have a homelab" becomes "my homelab does things for me." It's also the point where you start thinking about reliability and failure modes, which is the shift from hobbyist to practitioner. Post 01's section on agents was abstract. After this project it isn't.

---

### Project 5 — Wire up monitoring and alerts for something you run

**What it is:** Pick one of your running services and instrument it properly. Uptime Kuma is a self-hosted uptime monitor — you point it at your services' URLs and it pings them on a schedule, then notifies you (Telegram, email, ntfy, or others) when something goes down. That's the floor. The ceiling is a proper metrics stack — Prometheus, Grafana, and node-exporter — which gives you CPU, memory, and disk graphs over time.

Start with Uptime Kuma. Add a metrics stack when you're curious why something is slow.

**What you'll learn:** What "observability" means in practice: the difference between knowing something is down (uptime monitoring) vs. knowing why it's slow (metrics). How to configure alerting that's useful without being spammy — alert on what needs human attention, not everything that twitches. And the uncomfortable lesson that services fail in ways you didn't anticipate, and you only know because you were watching.

**Rough effort:** Uptime Kuma is a single Docker container and fifteen minutes of configuration. The full metrics stack is a weekend. Start with the easy version and let curiosity pull you further.

**Why it's a good rung:** This teaches "production thinking" — the mindset shift where you're not just running services but running services reliably. Every serious homelab eventually gets a monitoring layer, and the people who add it after their second mystery outage wish they'd added it after their first. If you're building anything others depend on, this isn't optional.

---

<div class="callout" markdown="1">
**Start here if you only do one.** Project 1 — Ollama plus a chat UI. It's achievable in an evening, it costs nothing after your hardware, and it makes everything from post 01 click. Once you have a local model running, you can use it as the backend for every other project on this list.
</div>

---

## General advice before you start building

**Start smaller than you think you need to.** The temptation is to plan the full system — monitoring, agents, RAG, the works — and build it all at once. This leads to a half-assembled pile that lives in Docker Compose limbo for weeks. Pick one project. Finish it. Then pick the next.

**Snapshot before you tinker.** This was in post 03 and it's worth repeating here because you're about to actually need it. Before you upgrade a container, change a config, or "quickly try something," take a snapshot. The five seconds this costs is the best insurance you have. The restore you do at 11 PM on a Tuesday will be the fastest you've ever appreciated a past decision.

**Keep a notes file.** Sounds obvious. Almost nobody does it until they've forgotten what they did to fix something the first time and spent two hours redoing it. A plain text file in your home directory, a note in Obsidian, a private GitHub repo — the format doesn't matter. Write down what you installed, what config you changed, and why it didn't work before the thing that made it work. Future you will be embarrassingly grateful.

**Don't expose anything to the internet without thinking.** Post 04 covered this, but it bears a callback here: the projects above are designed to live on your local network, behind your reverse proxy, accessible only to you. The moment you poke a hole in your firewall or set up an external tunnel, you've changed the threat model. That's sometimes fine and sometimes not — but it's a decision to make deliberately, not accidentally.

---

<div class="sidebar-note" markdown="1">
**What I actually built first.** My first project was a local chat UI backed by a small language model — nothing fancy, just proof that the thing could answer questions from my own machine without touching a cloud API. The second thing I built was a service I actually use: a bookmark manager that sits behind a local subdomain. I visit it every day, which means I'd notice immediately if it broke. That accountability changed how I thought about reliability. The monitoring came later, after one too many "wait, when did that go down?" moments.
</div>

---

## That's the arc

Six posts ago — [post 00]({% post_url 2026-06-05-00-start-here %}) — you had a laptop and a vague idea about AI homelabs. Since then you've covered how language models actually work, chosen hardware, built a VM, locked it down, and now you have a ladder of real projects to climb.

That's not nothing. Most people who say they want to run local AI never get past "I should look into that."

The homelab mindset is: ship something small, break it, fix it, understand it better than before. Repeat. The projects above are designed to be rungs, not destinations. After project 5 you'll be ready to ask your own questions — "what if I wired the RAG setup into the agent pipeline?" — and you'll have the vocabulary and the infrastructure to find out.

The best thing about running your own stack is that no one's rate-limiting you, no one's reading your prompts, and if something breaks you get to be the one who fixes it. That's the whole deal. Keep climbing.

---

*Previous post: [Securing the Box]({% post_url 2026-07-03-04-secure-the-box %}) — Series start: [Start Here]({% post_url 2026-06-05-00-start-here %})*
