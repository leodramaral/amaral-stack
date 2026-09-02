# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal bilingual (pt-BR / en) blog and mini-CV site for Leandro Amaral, built with Hugo and the `enervoid` theme. `.gitmodules` declares `themes/enervoid` as a submodule, but the current `HEAD` has it committed as regular tracked files (`git ls-tree` shows a plain `tree`, not a `160000` gitlink) — so `git submodule update --init --recursive` is a harmless no-op, not a real requirement; the theme is already there after a normal clone. No JS/CSS build pipeline — there is no `package.json`; Tailwind CSS v4, devicon, and Font Awesome are all loaded from CDNs in `<head>`.

**Hugo version matters:** this repo's `layouts/` use the `_partials` / `_markup` lookup convention, which requires Hugo Extended **0.146.0+** (see Deployment below). An older Hugo (e.g. a distro-packaged 0.123.x) fails with errors like `partial "head.html" not found` even though the file exists — that's a version mismatch, not a missing file.

## Commands

- `hugo server` — local dev server with live reload. Requires Hugo Extended 0.146.0+ on `PATH`.
- `hugo --minify --baseURL "<url>/"` — production build (mirrors `.github/workflows/deploy.yml`), outputs to `public/`.
- `hugo new content br/blog/<slug>/index.md` (and the `en/` equivalent) — create a new post from `archetypes/default.md`. Posts are leaf bundles: `index.md` plus any images live together in the post's directory.
- There is no lint/test suite in this repo.

## Deployment

`.github/workflows/deploy.yml` builds with Hugo Extended 0.146.0 and deploys `public/` to GitHub Pages on every push to `main`.

## i18n

- Two languages configured in `hugo.toml`: `br` (default, `content/br`) and `en` (`content/en`). Each post exists as a matching pair under both content trees (e.g. `content/br/blog/usando-groq-com-sdk-openai/` ↔ `content/en/blog/using-groq-with-openai-sdk/`) — Hugo pairs translations by matching directory/file name within each language's content dir.
- UI strings (not post content) are translated via `layouts/i18n/br.toml` / `en.toml` and pulled into templates with `{{ T "key" }}`.
- The language switcher in `header.html` walks `.AllTranslations` to link to the current page's counterpart in the other language.

## Layout override pattern

Hugo resolves templates from the site's `layouts/` before falling back to `themes/enervoid/layouts/`. Nearly every file this site customizes is a same-path override of a theme file, e.g.:

- `layouts/_partials/header.html`, `home.html`, `article/{meta,share,related}.html`, `head/opengraph.html` — site branding, social links, share buttons, related-posts logic.
- `layouts/blog/single.html` — adds the sticky table-of-contents sidebar around the theme's article rendering.
- `layouts/_markup/render-heading.html` — custom heading render hook for the underline-on-hover anchor style.

When changing look-and-feel, check whether the theme already provides the partial (`themes/enervoid/layouts/...`) before writing a new one — most changes should be overrides, not new templates.

## Styling & theming

- Dark mode is the default look; `assets/css/theme.css` (processed via Hugo Pipes in `head/css.html`) supplies overrides layered on top of the theme's Tailwind utility classes, plus a `[data-theme="light"]` block for light mode.
- The theme toggle button in `header.html` sets/removes `data-theme="light"` on `<html>`, persists the choice to `localStorage`, and dispatches a `theme-changed` event (Mermaid re-renders in response, since Mermaid needs a matching theme).
- Page transitions use Swup (loaded in the theme's `head/js.html`); elements needing fade animation use classes matching `swup-transition-*`.

## Content authoring notes

- Post front matter fields in use: `title`, `date`, `author`, `tags`, `description` (also feeds Open Graph/Twitter meta via `head/opengraph.html`, falling back to `.Summary` when absent), and optionally `image` (relative to the page bundle) or `site.Params.og_image`.
- "Related posts" (`article/related.html`) are computed by intersecting `.Params.tags` against other pages under the same tag taxonomy — there's no manual linking.
- Mermaid diagrams work via a fenced code block with the `mermaid` language tag (handled in the theme's `render-codeblock.html`); DBML/ER-diagram support was discussed in `content/br/construcao/development-plan.md` but was never actually implemented — don't assume a DBML render hook exists.
- `content/br/construcao/*` are planning documents (`build.render/list: never`), not published pages — they record the original spec and phased plan for how this site was built, useful for context on *why* something looks the way it does.
