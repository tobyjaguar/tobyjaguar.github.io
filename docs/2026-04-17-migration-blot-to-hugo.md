# Migration: Blot.im to Hugo + GitHub Pages

**Date:** 2026-04-17

## Motivation

The blog was hosted on Blot.im at $4/month. Given minimal traffic, the goal was to move to a free hosting solution while preserving all content. GitHub Pages was chosen since the repo was already on GitHub.

## Framework Decision

**Jekyll** was the initial choice (native GitHub Pages support, no CI needed), but Ruby version conflicts made it impractical — the system Ruby (2.6) was too old for the `github-pages` gem, and the Homebrew Ruby (4.0) was too new for Jekyll 3.9. Since CI/CD was needed anyway, we pivoted to **Hugo**:

- No runtime dependencies (single Go binary)
- Builds in ~150ms
- GitHub provides an official Pages workflow for Hugo
- PaperMod theme chosen for clean, minimal design

## What Was Migrated

- **17 blog posts** (2019–2024) from `Posts/YYYY/PostName/index.md` to `content/posts/YYYY-MM-DD-slug.md`
- **3 pages** (about, contact, projects) from `Pages/` to `content/`
- **~58 post images** from alongside each post's `index.md` to `static/assets/posts/<slug>/`
- **16 site-wide assets** to `static/assets/`
- **Brave Rewards verification** file to `static/.well-known/`

## Content Transformations

- Blot metadata (`Title: X`, `Date: MM/DD/YYYY`) converted to YAML frontmatter
- Dates converted from `MM/DD/YYYY` to `YYYY-MM-DD`
- Image paths rewritten from relative (`_img.jpg`, `./_img.jpg`) to absolute (`/assets/posts/slug/_img.jpg`)
- Page asset paths rewritten from `../assets/` to `/assets/`
- `target="new"` replaced with `target="_blank" rel="noopener noreferrer"`

## New Structure

```
tobyjaguar.com/
  hugo.toml                          # Site config
  content/
    posts/YYYY-MM-DD-slug.md         # Blog posts (17)
    about.md, contact.md, projects.md # Pages
  static/
    assets/                          # Site-wide images
    assets/posts/<slug>/             # Per-post images (17 dirs)
    .well-known/                     # Brave Rewards
    CNAME                            # Custom domain
  themes/PaperMod/                   # Theme (git submodule)
  .github/workflows/hugo.yml         # CI/CD
  docs/                              # Project docs (not rendered)
  CLAUDE.md                          # Claude Code guidance (not rendered)
```

## Progress

### Completed (2026-04-17)
- [x] All content migrated to Hugo (17 posts, 3 pages, ~58 images)
- [x] PaperMod theme added as git submodule
- [x] GitHub Actions workflow created (`.github/workflows/hugo.yml`)
- [x] Responsive hero image added to home page (`tobyjaguar_landscape.jpeg`)
- [x] CrossWorlds project added to projects page as top entry
- [x] Blot push URL removed from origin remote
- [x] GitHub repo renamed from `tobyjaguar.com` to `tobyjaguar.github.io`
- [x] Local origin updated to `git@github.com:tobyjaguar/tobyjaguar.github.io.git`
- [x] GitHub Pages enabled with `build_type: workflow` (GitHub Actions)
- [x] DNS A records set to GitHub Pages IPs (`185.199.108-111.153`)
- [x] DNS CNAME `www` set to `tobyjaguar.github.io`
- [x] Custom domain configured in GitHub Pages settings

- [x] First push to `master` — GitHub Actions build succeeded
- [x] DNS propagated to GitHub Pages IPs
- [x] HTTPS / SSL certificate active
- [x] Site live at `https://tobyjaguar.com`

### Remaining
- [ ] Cancel Blot subscription

### Migration complete: 2026-04-17
