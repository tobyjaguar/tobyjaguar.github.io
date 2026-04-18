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

## Current Status

Site is live. Migration from Blot.im to Hugo + GitHub Pages completed 2026-04-17.

- Site: `https://tobyjaguar.com` (custom domain with HTTPS)
- Repo: `tobyjaguar/tobyjaguar.github.io`
- Remaining: Cancel Blot subscription
