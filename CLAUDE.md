# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based personal tech blog using the "Satellite" theme, deployed via GitHub Pages. The site is written in Korean and English, covering Vue.js development, tech memoirs, and development guides.

## Common Commands

```bash
# Install dependencies
bundle install

# Run local development server (available at http://localhost:4000)
bundle exec jekyll serve

# Build the site without serving
bundle exec jekyll build
```

## Architecture

### Directory Structure

- `_pages/` - Blog posts and content organized hierarchically by category
  - Each subdirectory represents a category (e.g., `Vue/`, `Memoir/`)
  - Every directory must contain an `index.md` file with front matter (`---\n---`)
- `_layouts/` - Page templates (`default.html`, `page.html`)
- `_includes/` - Reusable HTML components (sidebar, navigation, search, post rendering)
- `_sass/` - SCSS stylesheets (supports dark/light mode via `darkmode.scss`)
- `assets/` - Static files (CSS, JS, fonts, images)
- `docs/` - Documentation (excluded from build)

### Content Creation

Blog posts are markdown files (`.md`) with required front matter:
```yaml
---
title: "Post Title"
date: "YYYY-MM-DD"
tags:           # optional
  - tag1
  - tag2
thumbnail: "/path/to/image"  # optional
bookmark: true              # optional - shows in sidebar nav
---
```

### Configuration

- `_config.yml` - Site settings, social links, external APIs (giscus comments, goatcounter analytics)
- Uses Jekyll plugins: `jekyll-feed`, `jekyll-sitemap`, `jekyll-seo-tag`, `jekyll-loading-lazy`
- Kramdown with GFM (GitHub Flavored Markdown) and hard line breaks enabled
