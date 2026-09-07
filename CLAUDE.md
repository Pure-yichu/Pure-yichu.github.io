# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Jekyll blog built on the [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) theme (gem `jekyll-theme-chirpy ~> 7.4`), deployed to GitHub Pages at https://pure-yichu.github.io. Posts are written in Chinese; most work in this repo is authoring posts.

## Commands

Ruby/Bundler are **not installed on this Windows host**. Options: reopen in the devcontainer (VS Code "Reopen in Container" — ships a Jekyll toolchain), or install Ruby 3.3 locally and run `bundle install` (`Gemfile.lock` is gitignored).

- Serve locally with live reload: `bash tools/run.sh` (flags: `-p` production env, `-H <host>`; auto-adds `--force_polling` inside Docker). Equivalent: `bundle exec jekyll s -l`
- Build and verify: `bash tools/test.sh` — production build into `_site/`, then runs `html-proofer` (`--disable-external`). This is the entire test suite; there are no unit tests.
- VS Code tasks "Run Jekyll Server" and "Build Jekyll Site" wrap these two scripts.

## Architecture

- **Theme lives in the gem.** Only `_config.yml`, `_plugins/`, `_tabs/`, `_data/`, `index.html`, and `assets/` are in-repo; layouts, includes, and Sass live inside the `jekyll-theme-chirpy` gem (locate via `bundle info --path jekyll-theme-chirpy`). To customize a layout or include, copy it out of the gem rather than writing one from scratch.
- **The whole repo is an Obsidian vault** (`.obsidian/` is at the repo root and committed). The obsidian-git plugin auto-commits a "vault backup" every few minutes — frequent auto-commits are normal, and every push to `main` triggers a site deploy. Obsidian's attachment folder is set to `assets/img/`. Posts are `_posts/YYYY-MM-DD-<title>.md` with `title` / `date` (+0800 offset) / `categories` / `tags` front matter; permalinks are `/posts/:title/`. Obsidian wikilinks (`![[...]]`) do not render on the site: reference images as `![alt](file.png)` (bare filename) plus `media_subpath: /assets/img/` in front matter — the theme prepends it automatically.
- **`_plugins/posts-lastmod-hook.rb`** injects `last_modified_at` into each post by shelling out to `git log`. It requires full git history — the CI workflow checks out with `fetch-depth: 0` for this reason. Do not prune history.
- **Deploy is CI-only**: `.github/workflows/pages-deploy.yml` runs on push to `main`/`master` — Ruby 3.3 setup, production Jekyll build, html-proofer, then Pages artifact deploy. `.nojekyll` is present because the site is built by Actions, not by GitHub Pages itself.
- **`assets/lib` submodule** (chirpy-static-assets) is intentionally uninitialized locally, and CI has `submodules: true` commented out. It is only needed if `assets.self_host.enabled` is set in `_config.yml` (currently off) — don't initialize it otherwise.
- **`code/`** is scratch space (a C++ `test.cpp`), unrelated to the site but tracked in the repo.

## Config notes

- `_config.yml`: keep `url: https://pure-yichu.github.io`; `baseurl` is empty; timezone `Asia/Shanghai`; PWA enabled; `paginate: 10`; tabs and archive permalinks are defined via the `tabs` collection and `jekyll-archives`.
- `_data/contact.yml` drives the sidebar contact icons; `_data/share.yml` drives post share buttons.
- Site images live in `assets/img/` (the sidebar avatar `1.jpg` and pasted post images from Obsidian).
