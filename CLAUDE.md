# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal Jekyll site (`ivictivi.github.io`) for Nacho Garcia, built on the **devlopr** theme (v0.5.0). It is a static, database-free site: all content (blog posts, authors, pages) is generated from Markdown and YAML files. Blog content is primarily in Spanish and focused on data engineering / Apache Airflow.

## Commands

```bash
bundle install                          # install Ruby gem dependencies
bundle exec jekyll serve --livereload   # dev server at http://localhost:4000
bundle exec jekyll build                # build static site into ./build
bundle exec jekyll build --future       # build including future-dated posts (used by CI)
bundle audit                            # scan dependencies for vulnerabilities
```

Docker alternatives:

```bash
docker-compose -f docker-compose-dev.yml  up --build --remove-orphans   # dev (watch mode), :4000
docker-compose -f docker-compose-prod.yml up --build --remove-orphans   # prod via nginx, :80
```

There is no test suite. CI validates the build (Jekyll Docker container) and runs markdownlint on `_posts/**/*.md`.

## Critical gotchas

- **Build output is `./build`, not the default `_site`.** This is set via `destination:` in `_config.yml`. Docker prod and nginx also expect `./build`.
- **Ruby is pinned to 3.3.4** (`.ruby-version`).
- **This is standalone Jekyll 4.x, NOT the `github-pages` gem.** GitHub Pages' native build will not work; deployment is handled by a custom GitHub Action instead (see below).
- **`_config.yml` changes require a server restart** — livereload does not pick them up.
- **`jekyll-admin` is intentionally disabled** in both `Gemfile` and `_config.yml` because it breaks the gh-pages build. Do not re-enable without understanding this.

## Deployment

- Deploy target is controlled by the plain-text `DEPLOY_STRATEGY` file. Valid values: `none`, `gh-pages` (current), `firebase`.
- `.github/workflows/deploy.yml` runs on push to `master` (and manual dispatch): it reads `DEPLOY_STRATEGY` and runs the matching job. The `gh-pages` job uses the `sujaykundu777/jekyll-deploy-action`.
- `CNAME` maps the site to the custom domain `ivictivi.com`.

## Architecture

The site is driven by Jekyll's convention-over-configuration layout plus heavy use of `_config.yml` for personalization.

- **`_config.yml`** is the primary control surface. Beyond standard Jekyll settings it holds author identity, work experience, education, projects, social usernames, feature toggles (`show_author_*`), pagination, and integration keys (Formspree, Olvy, Disqus/Hyvor, Snipcart, etc.). Most homepage/about content is edited here rather than in HTML.
- **`_layouts/`** — page templates. `default.html` is the root wrapper (it applies the `compress` layout for HTML minification and injects header/footer/newsletter). Content layouts (`home`, `post`, `blog`, `about-me`, `contact`, `page`, `full-width`, `author`) build on it.
- **`_includes/`** — reusable partials assembled by layouts, e.g. `head.html`, `header.html`, `footer.html`, `hero.html`, the `author_*` skill/bio blocks, `blog_*` post components, and social follow buttons. Change shared UI here, not in individual pages.
- **`_sass/_devlopr.scss`** is the single stylesheet source, compiled compressed (`sass.style: compressed`). `assets/` also contains prebuilt vendor libs under `bower_components/` plus `css/`, `js/`, `img/`.
- **Collections** (`_config.yml`): `authors` (output to `/blog/authors/:slug`) and `products` (e-commerce via Snipcart).

## Content conventions

- **Blog posts** live in `_posts/` named `YYYY-MM-DD-title.md`. Front matter uses `layout: post`, `title`, `author` (must match an author key), `categories`, and `tags`. Example:

  ```yaml
  ---
  layout: post
  title: "Apache Airflow 3.0: Novedades y Características"
  author: ivictivi
  categories: [airflow, data engineering, open source]
  tags: [airflow, airflow3, data pipeline, orchestration]
  ---
  ```

- **Authors are defined in two places** and must be kept in sync: `_data/authors.yml` (used by post bylines) and the `_authors/<username>.md` collection file (generates the author page). The current author key is `ivictivi`.
- **Top-level pages** (`about.md`, `contact.md`, `index.html`, `404.md`) are Markdown/HTML with front matter that sets `layout` and `permalink`.
- **`categories/`** holds one Markdown file per category listing page.
- **Content editing via CMS**: `admin/config.yml` configures a Netlify CMS backend (GitHub-based, `editorial_workflow`) targeting the `_posts`, `_pages`, and `_layouts` folders. Media uploads go to `assets/img/posts`.
