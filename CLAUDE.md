# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal blog/portfolio site for Nacho Garcia (ivictivi), built on the **devlopr** Jekyll theme (v0.5.0, by Sujay Kundu). Content is authored in **Spanish**. All content (blog posts, authors, categories) is generated from Markdown/YAML files — there is no database. Although devlopr ships as a gem, this repo vendors the full theme source locally (`_layouts`, `_includes`, `_sass`), so edit those files directly rather than expecting theme-gem overrides.

## Commands

Requires **Ruby 3.3.4** (pinned in `.ruby-version`).

```sh
bundle install                         # install gems
bundle exec jekyll serve --livereload  # local dev at http://localhost:4000
bundle exec jekyll build               # production build (output -> ./build)
bundle exec jekyll build --watch       # rebuild on change
bundle audit                           # dependency vulnerability scan
```

Docker alternative:

```sh
docker-compose -f docker-compose-dev.yml  up --build --remove-orphans   # dev, http://localhost:4000
docker-compose -f docker-compose-prod.yml up --build --remove-orphans   # prod (nginx)
```

There is no test suite. `.github/workflows/markdownlint-config.json` configures Markdown linting.

## Critical gotchas

- **Build output is `./build`, not the Jekyll default `_site`.** Set via `destination: ./build` in `_config.yml`. `netlify.toml` publishes `build`. Don't assume `_site`.
- **Default branch is `master`**, not `main`. The deploy workflow and Netlify CMS backend both target `master`.
- **`jekyll-admin` is intentionally disabled** in both `Gemfile` and `_config.yml` because it breaks GitHub Pages builds. Do not re-enable without understanding that trade-off.
- The `CNAME` (`ivictivi.com`) and the `url`/author URLs in `_config.yml` (`ivictivi.github.io`) intentionally differ.

## Deployment

Deployment is driven by GitHub Actions (`.github/workflows/deploy.yml`) on push to `master`. The target is read from the **`DEPLOY_STRATEGY`** file (a single word), which must be one of `gh-pages`, `firebase`, or `none`. **Currently `gh-pages`.** To change hosting target, edit that file — do not hardcode a target in the workflow. Firebase deploys require `FIREBASE_TOKEN` and `FIREBASE_PROJECT_ID` repo secrets.

## Architecture

Standard Jekyll data-flow, with these repo-specific conventions:

- **`_config.yml`** is the control panel: author identity/bio, work experience, education, projects, social links, and feature toggles (`show_author_*`) are all data here, consumed by `_includes` partials. Site-wide feature integrations (Snipcart ecommerce, Disqus/Hyvor comments, Algolia search, Mailchimp, Formspree/Getform) are configured here too.
- **`_layouts/`** — page templates. `default.html` wraps everything (via `compress.html` for HTML minification) and injects header/footer/newsletter + Snipcart. Content layouts available for front matter `layout:` — `home`, `post`, `blog`, `about-me`, `contact`, `author`, `page`, `full-width`.
- **`_includes/`** — composable partials (hero, header, footer, author skills, follow buttons, comments, share, newsletter). This is where most presentational markup lives.
- **`_posts/`** — blog posts named `YYYY-MM-DD-title.md`. Front matter requires `layout: post`, `title`, and `author` (the `author` value **must** match a key in `_data/authors.yml`). `categories`/`tags` are arrays.
- **`_authors/` + `_data/authors.yml`** — author profiles. Both must be kept in sync: `_data/authors.yml` powers post attribution; `_authors/*.md` are rendered as author pages (collection `authors`, permalink `/blog/authors/:slug`).
- **`categories/`** — one Markdown page per category; `all.md` auto-lists every category via `site.categories`.
- **`_sass/_devlopr.scss`** is the single Sass entrypoint, compiled compressed (`sass.style: compressed`) into `assets/css`. Front-end vendor libs live in `assets/bower_components` (jQuery, isotope, lightgallery, font-awesome, etc.).
- **`admin/`** — Netlify CMS (`admin/config.yml`), GitHub backend on `ivictivi/ivictivi.github.io@master`, editorial workflow enabled. Uploaded media goes to `assets/img/posts`.

## Adding a blog post

Create `_posts/YYYY-MM-DD-slug.md` with front matter: `layout: post`, `title`, `author` (existing key from `_data/authors.yml`), and optionally `categories: [..]` / `tags: [..]`. Body is Kramdown Markdown (Rouge syntax highlighting). Pagination shows 4 posts per page under `/blog/page/:num/`.
