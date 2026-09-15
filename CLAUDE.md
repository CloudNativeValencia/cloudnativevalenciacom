# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Static marketing site for the Cloud Native Valencia community (cloudnativevalencia.com), plus a Hugo-generated blog mounted at `/blog/`. No build tooling for the main site — it's hand-written HTML/CSS/vanilla JS deployed as-is by Netlify.

## Repo layout

- `index.html`, `styles.css`, `main.js` — the entire main site (single page, section-anchored nav: `#videos`, `#cfp`, `#about`, `#speakers`, `#organizers`, `#sponsors`).
- `hugo-blog/` — Hugo source for the blog (`content/posts/`, `layouts/`, `hugo.toml`). Netlify builds this on every deploy; its output is **not** committed.
- `blog/` — Hugo's build output, served at `/blog/`. Gitignored and regenerated on each deploy — never hand-edit files here.
- `speaker-badge/index.html` — standalone microsite/page, independent of the main site's CSS/JS.
- `netlify.toml` — build command, security headers, cache rules, and a hard block redirecting `/hugo-blog/*` to 404 (keeps Hugo source unreachable in production).
- `_redirects` — simple path redirects (e.g. `/rsvp` → an external CNCF community event page).

## Build & run locally

No package manager, no `npm install`. Two independent things can be served:

- **Main site**: any static file server from repo root, e.g. `npx serve .` or `python3 -m http.server`.
- **Blog**: from `hugo-blog/`, run `hugo server` for live-reload dev, or replicate the production build with:
  ```bash
  cd hugo-blog && hugo --cleanDestinationDir --minify --baseURL "https://cloudnativevalencia.com/blog/"
  ```
  This writes to `../blog` (per `publishDir` in `hugo.toml`). Pin/match the Hugo version in `netlify.toml`'s `HUGO_VERSION` when testing locally.

There is no test suite and no linter configured in this repo.

## Deployment

Netlify builds on push to `main`: runs the Hugo command above, then publishes the whole repo root (`publish = "."`), so the hand-written HTML/CSS/JS and the freshly built `blog/` ship together in one deploy. Every PR gets a Netlify deploy preview automatically.

## Blog content workflow

Posts are Markdown files in `hugo-blog/content/posts/`, filename = URL slug (`permalinks.page.posts = "/:contentbasename/"`, so `my-post.md` → `/blog/my-post/`). Required front matter:

```yaml
---
title: "..."
date: YYYY-MM-DD
description: "..."
authors: ["Name"]
draft: false
tags: ["Category"]
---
```

`authors` and `tags` are the only two taxonomies (`hugo.toml`); no categories. The intended external-contributor flow (see README.md) is fork → add file under `content/posts/` → PR → Netlify deploy preview → merge — no local Hugo install required for that path.

## Things to know before editing

- **CFP and Sponsor CTAs point to external services**, not a local form: `https://sessionize.com/cloud-native-valencia-2026` for talk submissions and GitHub Sponsors for sponsorship. Older docs in this repo (`README.md`, `DEPLOYMENT.md`, `PROJECT_SUMMARY.md`) describe an earlier iteration with a Netlify Forms CFP modal and dynamic YouTube API grid — that no longer matches `index.html`/`main.js`. Trust the code over those docs.
- `main.js`'s `CONFIG` object at the top holds the YouTube channel handle and scroll-behavior tuning (sticky-button threshold, reveal-animation thresholds) — check there before assuming behavior is hardcoded inline.
- Security headers, cache policy, and the Hugo source lockout are all centralized in `netlify.toml`; don't duplicate cache/security logic elsewhere.
- `assets/` mixes hand-placed images with generated thumbnails (e.g. `cncf-valencia-photo-thumbnail-*.jpg`, `gallery*.png`) — check existing naming/sizing conventions before adding new images, since there's no build step to optimize them.
