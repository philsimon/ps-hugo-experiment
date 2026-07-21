# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Hugo static site (`my-experiment`), deployed to GitHub Pages at https://philsimon.github.io/my-experiment/. Content lives in `content/posts/`; the active theme is `beautifulhugo` (`themes/beautifulhugo`), added as a git submodule.

## Commands

```bash
# Clone with submodules (themes won't populate otherwise)
git clone --recurse-submodules <repo>
# or, if already cloned:
git submodule update --init --recursive

# Local preview with live reload (writes dev output to public/, gitignored)
hugo server

# Local production-equivalent build, for checking output before pushing
hugo --minify
```

## Deployment

**Deploys are fully automated via `.github/workflows/hugo.yml` — push to `main` and it's live in about a minute.** GitHub Pages is configured with `build_type: workflow` (via `gh api repos/philsimon/my-experiment/pages`), meaning Pages serves whatever the Actions workflow uploads as an artifact, not any committed folder or branch.

- **`docs/` and `public/` are both gitignored and untracked.** They used to be committed build output — first `docs/`, then briefly both — which caused the site to go stale repeatedly whenever content changed but nobody remembered to rebuild and commit the HTML (this happened at least three times before the workflow was fixed). Do not re-add either directory to git; if you see them tracked again, something has regressed.
- **Never edit content directly on GitHub's web UI without also being aware CI will now handle the rebuild.** Before the Actions workflow existed, direct edits to `content/*.md` via the GitHub UI silently didn't update the live site, because nothing rebuilt the committed `docs/` folder. That failure mode is now gone — any push to `main`, from anywhere, triggers a full rebuild and deploy.
- If the workflow needs to be re-run without a new commit, use `workflow_dispatch` (it's enabled) via `gh workflow run hugo.yml`.

## Architecture

- **Themes are git submodules**, declared in `.gitmodules`. `themes/ananke` and `themes/hugo-book` (earlier themes tried this session) are still present but unused since the switch to `beautifulhugo`; either can be removed if they won't be revisited.
- **`beautifulhugo` expects `Params.mainSections` to match the actual content section name.** The theme's own example config sets `mainSections = ["post", "recipe"]`, but this site's content lives under `content/posts/` (plural), so `hugo.toml` sets `mainSections = ["posts"]`. Get this wrong and the homepage/RSS feed silently omit all posts — it's not a build error, just empty output.
- Some theme configs (e.g. Blowfish, tried and reverted) expect a `config/_default/*.toml` directory structure instead of a single `hugo.toml`. `beautifulhugo` uses the single-file style like the themes before it — don't mix the two approaches in the same build.
- **`baseURL` in `hugo.toml` is a local-dev fallback only** — the production workflow overrides it at build time with `--baseURL` set from `actions/configure-pages`, so it always matches whatever URL Pages actually assigns. Still keep the committed value pointed at `https://philsimon.github.io/my-experiment/` for local builds and previews.
- A `hugo server` process left running in the background will keep rewriting `public/` with dev-mode markup (injected livereload script, `noindex, nofollow` robots meta, unminified CSS refs). This no longer matters for deploys since `public/` isn't committed or published, but it can be confusing if you're inspecting `public/` output directly.
