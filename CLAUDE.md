# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio site for Robert Kovacs (robikovacs.com). Single-page Jekyll site deployed to GitHub Pages. No JavaScript frameworks — pure HTML/CSS with Liquid templating.

## Commands

```bash
# Install dependencies
bundle install

# Local development server (http://localhost:4000)
bundle exec jekyll serve

# Production build
JEKYLL_ENV=production bundle exec jekyll build

# Lint Ruby files
bundle exec rubocop
```

## Architecture

- **Single layout**: `_layouts/default.html` — the only layout template, includes SEO tags via `jekyll-seo-tag` plugin
- **Single page**: `index.html` — all content lives here (header, projects, social links)
- **Styling**: `assets/css/main.css` — CSS variables for dark/light mode theming, mobile breakpoint at 600px
- **No includes or collections** — this is intentionally minimal

## Build & Deploy

Push to `main` triggers `.github/workflows/jekyll.yml`: Ruby 3.4 + bundler cache, Jekyll build, deploy to GitHub Pages. No manual deploy steps.

## Key Config

- `_config.yml`: site title, URL (`https://robikovacs.com`), gravatar hash, plugins (`jekyll-seo-tag`, `jekyll-sitemap`)
- `.rubocop.yml`: all new cops enabled, line length max 120
- Build output goes to `_site/` (gitignored)
