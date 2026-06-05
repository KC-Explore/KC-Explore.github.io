---
layout: post
title: "Securing the Box: SSH & the Physical-Access Truth"
date: 2026-07-03
series: "Start Your AI Homelab"
part: 4
description: "Harden SSH the right way — then understand why physical access is the security boundary that actually matters for a homelab."
---

Security advice for homelabs usually falls into one of two failure modes: way too paranoid for what you're actually protecting, or dangerously hand-wavy because "it's just home stuff." This post tries to hit the honest middle.

We're going to harden SSH properly — it's cheap, correct, and you should just do it — and then I'm going to tell you the uncomfortable thing that most tutorials skip: none of that matters if someone can walk up and touch your machine.

Let's go in order.

---

## Part A — SSH Hardening

Your VM from [post 03]({% post_url 2026-06-26-03-prepare-the-vm %}) is running. Someone can SSH into it with a username and password. That's fine for a first boot. It stops being fine the moment anything on that machine is accessible from outside your LAN, and even on your LAN it's sloppy hygiene. Fix it now, before you build anything on top.

### 1. Key-Based Authentication

Passwords can be guessed, leaked, or phished. SSH keys cannot be guessed — they're mathematically intractable. This is the single highest-leverage change you'll make.

On your **local machine** (not the server):

```bash
# Generate an ed25519 keypair — modern, fast, short keys
ssh-keygen -t ed25519 -C "your-label-here"
# Accept the default path (~/.ssh/id_ed25519) or name it something meaningful
# Set a passphrase — this protects the key if your local machine is compromised
```

Copy the public key to your server:

```bash
ssh-copy-id your-user@labvm
# Prompts for your password one last time, then installs the key
```

Test it immediately in the **same terminal**:

```bash
ssh your-user@labvm
# Should log in without asking for a password
```

If it works, keep this session open. You'll need it in the next step.

### 2. Disable Password Auth and Root Login

Open `/etc/ssh/sshd_config` on the server:

```bash
sudo nano /etc/ssh/sshd_config
```

Find and set these lines (uncomment them if they're commented out):

```
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
```

**Before you restart sshd, open a second SSH session in a new terminal.** If you misconfigured something and lock yourself out, you want an existing session to fix it.

```bash
# In a NEW terminal window/tab, confirm you can still log in with your key:
ssh your-user@labvm
# If this works, you're safe. If it doesn't, don't restart sshd yet — go fix the config.
```

Once confirmed:

```bash
sudo systemctl restart ssh
```

Try a third connection from scratch. No password prompt means it worked.

### 3. Limit Exposure: Port and AllowUsers

**Change the default port.** This is not real security — anyone running a port scan finds it in seconds. It's friction. It cuts about 95% of the automated noise from bots that only hammer port 22, which keeps your logs readable. Call it what it is.

In `/etc/ssh/sshd_config`:

```
Port 2222
```

Pick any unprivileged port (1024–65535) that nothing else is using. Update your client's `~/.ssh/config` to match:

```
Host labvm
    HostName 10.0.0.50
    User your-user
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

**Restrict who can log in** with `AllowUsers`. Even if a user account exists on the server, only listed users can authenticate via SSH:

```
AllowUsers your-user
```

Restart sshd after any change. Always test from a second session first.

### 4. Firewall with ufw

A firewall that defaults to allowing everything is a firewall that isn't doing anything. Set the policy to deny everything, then punch only the holes you need.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH on whatever port you chose
sudo ufw allow 2222/tcp

# Allow any other services you're actually running and need access to
# sudo ufw allow 8080/tcp   ← example: a web UI you want reachable on LAN

sudo ufw enable
sudo ufw status verbose
```

Verify your SSH port is listed as ALLOW before you close your existing session. If you forget to allow SSH and enable ufw, you will lock yourself out. (The fix is to access the machine physically or via the hypervisor console — which is a preview of Part B.)

### 5. fail2ban

fail2ban watches your logs and temporarily bans IPs that fail authentication too many times. It won't stop a targeted attack, but it stops the automated credential-stuffing bots that run continuously across the entire internet.

```bash
sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban
```

The defaults are reasonable for SSH. If you want to tune the ban threshold:

```bash
sudo nano /etc/fail2ban/jail.local
```

```ini
[sshd]
enabled = true
port    = 2222
maxretry = 5
bantime  = 3600
findtime = 600
```

`maxretry = 5` means five failures within ten minutes (`findtime`) earns a one-hour ban (`bantime`). Adjust to taste. Check current bans:

```bash
sudo fail2ban-client status sshd
```

### 6. Keep It Off the Public Internet

This is worth more than all five steps above combined.

Do not port-forward SSH from your home router. Do not expose any homelab service directly to the internet unless you have a very specific reason and know exactly what you're doing. The moment your SSH port is internet-routable, you are fighting every automated scanner and credential bot on the planet. fail2ban slows them down. A VPN makes them irrelevant.

Use a mesh VPN — [Tailscale](https://tailscale.com/) is the easiest entry point, [WireGuard](https://www.wireguard.com/) if you want to run your own. The idea: your homelab machine and your laptop are connected through an encrypted overlay network. From the public internet's perspective, your SSH port doesn't exist. From your laptop's perspective, you SSH to a private mesh address as if you were on the same LAN.

Tailscale's free tier handles most homelabs comfortably. Setup takes about ten minutes:

```bash
# On the server
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# On your local machine — install the Tailscale client for your OS,
# then authenticate both devices to the same account.
# They'll get addresses in the 100.x.x.x range.

# SSH via the mesh address — no router port-forwarding needed
ssh your-user@100.x.x.x
```

Once this is working, there's no reason to ever expose SSH to the open internet. The attack surface collapses to: people who can authenticate to your Tailscale account. That's a much smaller problem.

---

<div class="sidebar-note" markdown="1">
**How I lock mine down.**

Keys only — no passwords anywhere. My SSH config file means I never type a hostname or port. My homelab VM is not accessible from the internet at all; I use a mesh VPN so connections look local regardless of where I'm connecting from. Remote access works the same whether I'm on my home network or somewhere else entirely, and the lab machine doesn't appear in any internet-facing context.

For anything that leaves the physical premises — a laptop, an external drive with backups — full-disk encryption is non-negotiable. Data at rest is encrypted before it goes anywhere I don't control. The lab box itself lives somewhere physically controlled; it's not in a common area where anyone could casually interact with it.

The firewall defaults to deny. Services are only reachable from the mesh network. Logs are monitored. That's it — nothing exotic, nothing clever. Boring and consistent beats sophisticated and occasionally skipped.
</div>

---

## Part B — The Physical-Access Truth

Now for the part that most SSH hardening guides bury in a footnote or skip entirely.

Everything you just did — key auth, disabled passwords, ufw, fail2ban, mesh VPN — protects you against **network-based** attacks. An attacker trying to reach your machine over the wire hits a wall.

**An attacker who can physically touch your machine doesn't care about any of it.**

---

<div class="callout" markdown="1">
**The uncomfortable truth.**

If someone has physical access to your machine, they own it. Not "they might be able to compromise it eventually." Immediately. They can boot from a USB drive, mount your filesystem, read every file, change passwords, install whatever they want, and leave no trace — in under ten minutes. Your SSH hardening is a locked front door on a house with no walls.

This isn't a failure of your configuration. It's how computers work. Software security assumes the hardware is trusted. The moment that assumption breaks, everything built on top of it breaks too.

For most homelabbers, the realistic threat model is: a curious roommate, a visiting friend who wanders into your office, a theft. Not a nation-state. Not a targeted corporate adversary. Right-size your response to what you're actually protecting against.
</div>

---

### What Physical Access Actually Lets Someone Do

Here's the concrete attack, step by step, so it's not abstract:

1. Boot from a USB drive (any live Linux distro)
2. Mount your server's disk — it appears as a regular volume
3. `chroot` into it, run `passwd your-user`, pick a new password
4. Reboot normally, log in with the new password
5. They now have full access with your account

Alternatively: pull the drive, put it in a USB enclosure, plug it into their own machine. Read everything. Copy everything. Walk out.

This is not an exotic technique. It requires no special knowledge. It's in every security curriculum as "basic physical access attack."

### The Actual Mitigations

**Full-Disk Encryption (LUKS)**

This is the real answer. If the disk is encrypted, pulling the drive or booting from USB gets an attacker a pile of ciphertext. Without the passphrase or key, it's useless.

On Ubuntu/Debian, LUKS encryption is offered during OS installation. Enable it. The cost is: you have to enter a passphrase each time the machine boots (or configure a key file, which has its own tradeoffs). For a homelab VM that doesn't reboot often, this is a minor inconvenience with meaningful protection.

If you didn't encrypt at install time, there's no painless retrofit — you'd need to reinstall. Worth knowing for your next build.

```bash
# Check if your current install is LUKS-encrypted:
lsblk -o NAME,FSTYPE,MOUNTPOINT | grep crypto
# or
sudo dmsetup ls --target crypt
```

If you see `crypto_LUKS` in the output, you're already encrypted. If not, file it away as "do this on the next machine."

**BIOS/UEFI Password and Boot Order**

Set a BIOS/UEFI password and configure the machine to boot only from its primary disk. This slows down the USB-boot attack — an attacker has to clear CMOS or disassemble the machine to get past it. Not impossible, but it adds friction and time.

On most motherboards this is in the UEFI settings under "Security" or "Boot." The exact menu varies by vendor. Look for "Supervisor Password" and "Boot Priority."

This is a speed bump, not a wall. Someone with enough time and physical access can bypass it. Combined with LUKS, though, you're raising the effort level considerably.

**Where the Box Physically Lives**

For a home setup, this is underrated. A server in a locked room is more secure than an internet-hardened machine sitting on an open desk in a common area. Locks and doors are physical security controls, and they work.

You don't need a cage or a data center. A spare room with a lock, a closet, an office you control — all of these reduce your exposure to the realistic threat model (roommate curiosity, visitors, opportunistic theft) significantly. A door lock costs zero dollars and takes no configuration.

**The Honest Takeaway**

Harden SSH because it's cheap, correct, and good hygiene. You should do it regardless. But be honest with yourself about what it protects: your network perimeter, not your hardware.

If you're running a homelab with data you actually care about — credentials, personal files, anything sensitive — full-disk encryption on anything that could leave your physical control is the right call. The box in your home office is probably fine sitting behind a locked door. The laptop you carry around, the external drive with backups, the NUC you'd take to a friend's place: encrypt those.

And if someone ever breaks in and takes the hardware, the disk encryption means they got hardware, not data. That's a bad day, but it's a recoverable one.

---

## Quick Reference: What Each Layer Buys You

| Control | Protects Against | Doesn't Protect Against |
|---|---|---|
| Key-based SSH auth | Password guessing, credential stuffing | Physical access |
| Disabled root login | Brute-force on root account | Physical access |
| ufw deny incoming | Port scanning, unsolicited connections | Physical access |
| fail2ban | Automated brute-force bots | Physical access |
| Mesh VPN (no port forwarding) | Internet-based attacks entirely | Physical access |
| LUKS full-disk encryption | Drive theft, USB boot attacks | Someone who has your passphrase |
| BIOS password + boot order | Quick USB boot attacks | Determined attacker with time |
| Physical location control | Opportunistic access | Targeted break-in |

The pattern is obvious. The software controls are all good and worth doing. Physical is where the game actually ends.

---

## Next Up

Post 05 gets back to building things: [Starter projects worth your first weekend]({% post_url 2026-07-10-05-starter-projects %}) — a short list of homelab projects that are genuinely useful day one, not just impressive on paper. Now that the box is hardened, we can actually put it to work.

---

*Previous: [Getting the VM ready]({% post_url 2026-06-26-03-prepare-the-vm %})*
