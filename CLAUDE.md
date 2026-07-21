# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Hugo static site (`my-experiment`), deployed to GitHub Pages at https://philsimon.github.io/my-experiment/. Content lives in `content/posts/`; the active theme is `hugo-book` (`themes/hugo-book`), added as a git submodule.

## Commands

```bash
# Clone with submodules (themes won't populate otherwise)
git clone --recurse-submodules <repo>
# or, if already cloned:
git submodule update --init --recursive

# Local preview with live reload (writes dev output to public/ by default)
hugo server

# Production build — GitHub Pages serves from docs/, NOT the Hugo default public/
hugo --minify --destination docs --cleanDestinationDir

# New post
hugo new content posts/my-post.md
```

## Architecture

- **Publish target is `docs/`, not `public/`.** GitHub Pages for this repo is configured (via repo Settings → Pages) to serve from the `main` branch, `/docs` folder — confirmed via `gh api repos/philsimon/my-experiment/pages`. Hugo's default output directory is `public/`, so every production build must pass `--destination docs`. Do not assume a plain `hugo` or `hugo --minify` call updates the live site.
- **`public/` is a leftover, not the deploy target.** It's tracked in git but not what Pages serves. Historically it has also picked up dev-server artifacts (see below), so treat modifications to it with suspicion — check whether they're a real content change or server noise before committing.
- **No CI currently builds or deploys this site.** A GitHub Actions workflow (`.github/workflows/hugo.yml`) was added in one commit and accidentally deleted two commits later in a commit titled "Build site" (likely an unintentional `git add -A` after a working-directory cleanup). Deploys are currently manual: run the production build command above, commit `docs/`, and push. If re-adding CI, don't reintroduce the same collision — a workflow should `git rm -r public` or gitignore it if it isn't the actual deploy source.
- **Themes are git submodules**, declared in `.gitmodules`. `themes/ananke` (the original theme) is still present but unused since the switch to `hugo-book`; it can be removed if it won't be revisited.
- **`hugo-book` is a documentation theme, not a blog theme.** Its default `BookSection` param only renders content under a `docs/` content section into the sidebar menu. This site sets `BookSection = "*"` in `hugo.toml` to render all content sections (i.e. `content/posts/`) into the menu without restructuring existing content. Keep this in mind if adding new content sections — they'll appear in the menu automatically under `*`.
- **`baseURL` in `hugo.toml` must match the real Pages URL** (`https://philsimon.github.io/my-experiment/`). It previously shipped with Hugo's placeholder `your-username.github.io` value, which got baked into every generated page (canonical links, OG tags, footer links) until corrected — a reminder that `baseURL` errors don't show up as build failures, only as bad links in the output.
- A `hugo server` process left running in the background will keep rewriting `public/` with dev-mode markup (injected livereload script, `noindex, nofollow` robots meta, unminified CSS refs) any time a watched file changes. If `git status` shows unexpected `public/` diffs, check for a stray `hugo server` process before assuming it's a real content change.
