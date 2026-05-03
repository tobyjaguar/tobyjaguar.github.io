# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal blog and portfolio site for Toby Jaguar, built with **Hugo** and the **PaperMod** theme. Deployed to **GitHub Pages** via GitHub Actions.

## Build & Dev Commands

```bash
hugo server -D      # Local dev server with drafts at http://localhost:1313/
hugo                 # Build static site to public/
```

Requires Hugo extended edition (`brew install hugo`).

## Deployment

Pushing to `master` triggers the GitHub Actions workflow (`.github/workflows/hugo.yml`) which builds and deploys to GitHub Pages. The custom domain `tobyjaguar.com` is configured via `static/CNAME`.

## Content Structure

- `content/posts/` — Blog posts as `YYYY-MM-DD-slug.md`
- `content/about.md`, `content/contact.md`, `content/projects.md` — Static pages
- `static/assets/` — Site-wide images (logos, avatars)
- `static/assets/posts/<slug>/` — Per-post images, one directory per post matching the post slug
- `static/.well-known/` — Brave Rewards verification

## Markdown Format

Posts use YAML frontmatter:

```yaml
---
title: "Post Title"
date: YYYY-MM-DD
---
```

Pages use `url` instead of `date`:

```yaml
---
title: "Page Title"
url: /slug/
---
```

## Working with Content

- New posts go in `content/posts/` named `YYYY-MM-DD-slug.md`
- Post images go in `static/assets/posts/<slug>/` and are referenced as `/assets/posts/<slug>/filename`
- Site config is in `hugo.toml`; theme is a git submodule in `themes/PaperMod`
- Raw HTML in markdown is enabled (`markup.goldmark.renderer.unsafe = true`)

## Non-Blog Files

Hugo only renders files inside `content/` and `static/`. Root-level files and other directories are ignored by the build and won't appear on the hosted site:

- `CLAUDE.md` — this file
- `docs/` — project documentation, progression notes, migration history

## Notes for Remote Agents

This repo may be edited by remote agents (e.g. the `hermes` agent on the Klawpheus server) that don't have a local Hugo install.

- **Don't preview locally.** There's no `hugo` binary on the agent's box. Trust the GitHub Actions build (`hugo.yml`) for verification — push, then check the run status.
- **After pushing**, confirm the latest run on `master` succeeded before reporting "deployed":
  ```bash
  gh run list --branch master --limit 1
  gh run watch        # if you want to block until it finishes
  ```
- **Push target.** Push directly to `master` for content-only edits (new posts, copy/typo fixes, image swaps). Open a PR instead for anything touching `hugo.toml`, `layouts/`, `themes/`, or `.github/workflows/` — those affect rendering or the build pipeline and deserve human review before they ship to production.
- **Submodule.** The PaperMod theme is a public git submodule at `themes/PaperMod`. On a fresh clone, run `git submodule update --init --recursive` (or clone with `--recurse-submodules`).
- **Commit identity.** Use a clearly-attributable identity for agent commits, e.g. `git config user.name "Hermes Agent"` and a `users.noreply.github.com` email, so it's obvious in `git log` which changes were agent-made.

## Current Status

Site is live. Migration from Blot.im to Hugo + GitHub Pages completed 2026-04-17.

- Site: `https://tobyjaguar.com` (custom domain with HTTPS)
- Repo: `tobyjaguar/tobyjaguar.github.io`
- Remaining: Cancel Blot subscription
