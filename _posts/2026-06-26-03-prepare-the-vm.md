---
layout: post
title: "Getting the VM Ready"
date: 2026-06-26
series: "Start Your AI Homelab"
part: 3
description: "Where your homelab actually lives: choosing a hypervisor, provisioning a base Linux VM, sizing it, snapshots, and first-boot setup."
---

If you followed [post 02]({% post_url 2026-06-19-02-choosing-your-stack %}), you picked a rough direction for your stack. Now you need somewhere to put it. That's what this post is about: giving your homelab a home that isn't your laptop's desktop folder.

The short version: you want a virtual machine. Everything else flows from that decision.

---

## Why a VM and not bare metal

The honest answer is reversibility. When you install things directly onto your laptop or a spare PC and something breaks, untangling it is tedious. When you do it inside a VM, you delete the VM and start over. That's the whole pitch.

More concretely:

- **Snapshots.** You can freeze the state of a VM before doing something risky and roll back to that exact point if it goes sideways. This is not a feature most operating systems expose natively. It's a superpower.
- **Isolation.** Your AI services don't share libraries or ports with your daily-driver environment. You can install conflicting versions of Python, Node, whatever, without caring.
- **Reproducibility.** If you ever want to rebuild the lab from scratch on better hardware, the process is just "spin up a new VM, follow the same steps." You're not reverse-engineering a years-old hand-rolled setup.

The mental model I find useful: your homelab should be *disposable and reproducible*. The VM is disposable. The knowledge and configs you build up are what you keep.

---

## Where to actually run the VM

You have three realistic options. None is universally correct.

### Option A: Your existing laptop or desktop, with VirtualBox (or similar)

VirtualBox is free, runs on Windows, macOS, and Linux, and is the lowest-friction way to start. Download a Linux ISO, create a VM, you're done in 20 minutes.

The catch: your laptop sleeps. Your homelab sleeps with it. If you just want to learn and experiment and don't care about services being up overnight, this is fine. If you want a Telegram bot or a scheduled scraper running at 2 AM, this will frustrate you quickly.

Use this option if you're not sure yet whether you want a homelab, or if you genuinely don't have other hardware.

### Option B: A spare PC or mini-PC running Proxmox (recommended)

This is the sweet spot. A mini-PC — the kind of small box that costs $150-300 used — draws 10-15W at idle, sits on a shelf, and runs 24/7 without complaint. You install Proxmox VE on it (type-1 hypervisor, runs directly on the hardware), then spin up one or more Linux VMs on top.

Why Proxmox specifically: it's free, mature, well-documented, and has a solid web UI for managing VMs without touching a command line. Snapshots, backups, and VM cloning are first-class features. It's also what most homelab guides assume, so you'll find help easily.

If you're in this for the long run and you have any spare hardware, this is where to land.

### Option C: A cloud VPS

If you have no spare hardware and don't want to buy any, a cheap VPS (a small virtual private server on a provider like Hetzner, DigitalOcean, or Vultr) gets you a 24/7 Linux box for $5-10/month. You can run everything in this series on it.

The trade-offs: it's not "at home," so it's less private. You're paying a recurring cost. And you can't do GPU passthrough later if you want to run local LLMs. It's a valid fallback, not the ideal destination.

---

<div class="sidebar-note" markdown="1">

**How my lab is laid out.** I have a small always-on box running a type-1 hypervisor. On top of that I have a handful of VMs carved up by purpose: one general-purpose VM for services and automation, one dedicated to security tooling, and a couple of lightweight containers for things like DNS and media. Before any significant change — upgrading a database, trying a new configuration — I take a snapshot. It takes about five seconds and has saved me at least a dozen times. Nothing in the lab shares state with my daily-driver machine.

</div>

---

## Provisioning a base Linux VM

For the rest of this series I'll assume Ubuntu Server LTS or Debian. Either works. Both are stable, well-supported, and have packages for everything you'll need.

**Step 1: Get the ISO.** Download the latest Ubuntu Server LTS from ubuntu.com, or Debian from debian.org. The server edition has no desktop environment — that's what you want. Less overhead, fewer attack surface.

**Step 2: Create the VM.** In Proxmox, click "Create VM" and walk through the wizard. In VirtualBox, click "New." The wizard will ask you for:

- Name: something you'll recognize. `labvm` or `homelab` is fine.
- Type/OS: Linux, Ubuntu or Debian as appropriate.
- Disk, RAM, CPU: see the callout below.
- Network: NAT works for VirtualBox. In Proxmox, use a bridged adapter so the VM gets its own IP on your LAN — this matters when you want to reach services from your phone or other devices.

**Step 3: Install the OS.** Attach the ISO, boot the VM, follow the installer. When it asks about additional packages during setup, skip them — you'll install what you need. The one thing to select during install: OpenSSH server. You want SSH access from day one.

---

<div class="callout" markdown="1">

**Sensible starting specs.** For a general-purpose services VM running the kinds of things in this series (automation tools, small web apps, scheduled scripts):

- **vCPU:** 2-4 cores
- **RAM:** 4-8 GB
- **Disk:** 40-60 GB

This is conservative and deliberately so. You can always expand later. What you can't do is shrink a disk easily, so don't overprovision on day one.

One caveat: local LLMs are a completely different beast. Running a model like Llama or Mistral locally needs 8-16 GB of RAM at minimum just for the model weights, and benefits enormously from a dedicated GPU. If that's your goal, plan for a separate VM (or a separate machine) with much larger specs — and possibly GPU passthrough if your hypervisor and hardware support it. Don't try to cram a local LLM into the same VM as your other services.

</div>

---

## First-boot setup

Fresh install, logged in, cursor blinking. Here's the checklist I run on every new VM before I do anything else:

```bash
# 1. Update everything
sudo apt update && sudo apt upgrade -y

# 2. Set your hostname (pick something meaningful to you)
sudo hostnamectl set-hostname labvm

# 3. Set your timezone
sudo timedatectl set-timezone America/Toronto
# Check available zones: timedatectl list-timezones | grep America

# 4. Create a non-root sudo user (if the installer didn't already)
sudo adduser yourname
sudo usermod -aG sudo yourname

# 5. Install essentials
sudo apt install -y curl git unzip htop

# 6. Confirm SSH is running (it should be from the install step)
sudo systemctl status ssh
```

That's it. A few minutes of work. The VM is now in a clean, known state with a real user account and the basics installed.

One thing worth emphasizing: **don't do your regular work as root.** The installer creates a root account; you should use a non-root user with `sudo` for everything day-to-day. It's a habit that prevents a surprising number of accidents.

---

## Take a snapshot right now

Before you install a single service or run another command, take a snapshot.

In Proxmox: select the VM, go to Snapshots, click "Take Snapshot." Give it a name like `clean-install`. Done.

In VirtualBox: with the VM powered off, go to Machine > Tools > Snapshots, click "Take." Same idea.

Why now, before anything is installed? Because this is the cleanest state the VM will ever be in. If anything goes wrong in the next post, the next month, or the next year, you can roll back to this snapshot and get a fresh start in seconds without downloading ISOs or running through the installer again.

I take a snapshot before any significant change: before upgrading a major package, before trying a new configuration, before installing something unfamiliar. It costs almost nothing and has saved me from painful rollbacks more times than I can count. Make it a reflex.

---

## What's next

You have a running VM, a non-root user, and a clean-install snapshot. The VM is reachable over your local network — but right now, anyone on that network (or anyone who can reach it) could potentially poke at it.

Before you install anything useful, you want to lock the front door. The next post covers securing this VM: setting up SSH key authentication, disabling password login, and understanding why SSH is how you'll actually drive this machine day-to-day (not the hypervisor console).

**Next up:** [Securing the box: SSH & the physical-access truth]({% post_url 2026-07-03-04-secure-the-box %})

---

*Previous: [Choosing your stack]({% post_url 2026-06-19-02-choosing-your-stack %})*
