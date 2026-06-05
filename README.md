# Start Your AI Homelab — blog series

A beginner-facing, first-person field guide to building an AI homelab, published as
a GitHub Pages (Jekyll) blog. The Markdown in [`_posts/`](_posts/) is the **single
source of truth** — Word `.docx` review copies are generated from it, never edited
separately.

## The series

| # | Post | What it covers |
|---|------|----------------|
| 00 | Start Here | What an AI homelab is, why bother, the baseline you need |
| 01 | AI foundations | Models, tokens, context, agents — in plain English |
| 02 | Choosing your stack | Subscriptions vs APIs vs local models; model landscape & trade-offs |
| 03 | Prepare the VM | Hypervisor, base Linux VM, snapshots, first boot |
| 04 | Secure the box | SSH hardening + why physical access is the real boundary |
| 05 | Starter projects | Five things to build first, easy → ambitious |

## Generate the Word review copies

```bash
# Highest quality (recommended): install pandoc once
sudo apt-get install -y pandoc
python3 build_docx.py        # writes docx/*.docx

# No pandoc? The builder falls back to python-docx automatically:
pip install python-docx
python3 build_docx.py
```

Output lands in `docx/` (gitignored). `00-MASTER-all-posts.docx` is the whole series
in one file for a single review pass.

## Preview the site locally (optional)

GitHub Pages builds Jekyll server-side, so this is only for local preview:

```bash
bundle install
bundle exec jekyll serve   # http://localhost:4000
```

## Publish

1. Set a real git identity (`git config user.name` / `user.email`).
2. Create the GitHub repo (`<username>.github.io` for a root site, or a project repo).
3. `git init && git add . && git commit && git push`.
4. Repo → Settings → Pages → build from the default branch.
5. Confirm the live URL renders all six posts and navigation works.

## Conventions

- Each post has a generic core plus a boxed **"How I run mine"** sidebar — anonymized,
  no real IPs / internal hostnames / full service inventory.
- Before any push, run the anonymization check:
  ```bash
  grep -rEn '192\.168\.|10\.|your-internal-hostname|your-tailnet' _posts/ index.md && echo "LEAK — scrub before publish" || echo "clean"
  ```
