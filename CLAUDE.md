# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Hugo static site blog hosted at [andornot.xyz](https://andornot.xyz), using the [hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack/) theme (managed as a git submodule). The site is deployed to GitHub Pages via GitHub Actions on push to `master`.

## Development Commands

```bash
# Local development server
hugo server

# Build for production (with garbage collection and minification)
hugo --gc --minify

# Create a new post
hugo new posts/my-post-title.md

# Create a new post with a page bundle (for posts with images)
hugo new posts/my-post-title/index.md
```

Before building locally, install the required Node.js dependencies (used by the theme's PostCSS pipeline):

```bash
npm install --dev @babel/cli @babel/core @babel/preset-env browserslist clipboard cssnano postcss-cli postcss-import postcss-mixins postcss-nested postcss-preset-env postcss-url
```

Hugo extended version is required (for SCSS/PostCSS support).

## Content Structure

- `content/posts/` — Blog posts. Simple posts are single `.md` files; posts with images use a **page bundle** (`content/posts/post-name/index.md` with images alongside).
- `content/page/` — Static pages (about, archives, books, links, search).
- `static/` — Static assets served at the root URL.
- `assets/img/` — Theme assets (e.g., avatar image).
- `config.yaml` — All site configuration including theme params, menus, comments, widgets.

## Post Front Matter

```yaml
---
title: "Post Title"
date: 2024-01-01T00:00:00+08:00
draft: false
tags: ["tag1", "tag2"]
categories: ["category"]
image: cover-image.jpg   # optional, for featured/OG image
---
```

The default content language is `zh-cn` (Chinese). Posts are primarily written in Chinese.

## Deployment

Pushing to `master` automatically triggers the GitHub Actions workflow (`.github/workflows/main.yml`), which builds the site and deploys it to the `gh-pages` branch. The custom domain `andornot.xyz` is set via a `CNAME` file written during the build step.

## Theme

The theme is at `themes/hugo-theme-stack` as a git submodule. To update it:

```bash
git submodule update --remote themes/hugo-theme-stack
```

Do not modify files inside `themes/` directly — override theme templates by creating matching files under `layouts/`.
