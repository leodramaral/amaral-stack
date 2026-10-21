# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal bilingual (pt-BR / en) blog and mini-CV site for Leandro Amaral, built with Hugo. The layout and visual language follow the DevDojo "Index" template (fixed identity sidebar, editorial post rows, Geist / Geist Mono, indigo accent). All templates live in this repo's `layouts/` — there is no theme: `themes/enervoid` was absorbed and removed (its MIT license is kept in `NOTICE-enervoid-LICENSE` because a few copied templates descend from it). No JS/CSS build pipeline — there is no `package.json`; Tailwind CSS v4 (browser build), devicon, Font Awesome and Google Fonts are all loaded from CDNs in `<head>`.

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
- UI strings (not post content) are translated via `i18n/br.toml` / `en.toml` and pulled into templates with `{{ T "key" }}`. Add every new key to both files.
- The language switcher (`_partials/lang-switch.html`, rendered from `header.html`) walks `.AllTranslations` to link to the current page's counterpart in the other language.
- **The pt-BR language key `br` is the ISO code for Breton.** Hugo localizes dates (`time.Format ":date_*"`, month/day names) by that key, so it renders Breton ("2 Gwen. 2026"). Never use `time.Format` for display dates: use `{{ partial "date.html" (dict "date" .Date "long" false) }}`, which builds them from the `month_N` / `month_short_N` i18n keys. Renaming the key would move `/br/` and every URL, so it stays.

## Layouts

- `baseof.html` — skip link, `header.html`, then the content panel (`lg:pl-72`, rounded bordered `bg-panel`) holding `#main`, the Swup container `#swup` and the footer. Every page uses the same `max-w-3xl` container.
- `_partials/header.html` — the whole site chrome in `#site-nav`: a fixed identity sidebar (`lg+`; avatar, `params.name` / `params.role`, `homepage_description`, menu, socials, language, theme toggle) and a sticky bar with a collapsible menu below `lg`. It also holds the one-time script for the burger menu and theme toggle (event delegation, so it survives Swup swaps). Social links come from `params.socials` (each needs `name`, `link`, `icon`).
- `_partials/menu.html` — menu from `[[languages.*.menus.main]]`; the icon comes from the entry's `params.icon` (`home` | `pencil` | `user`; add new SVG paths to the dict in the partial).
- `_partials/post-row.html` — the single editorial post row, used by the home page, blog list, section/term pages and related posts. The title link is stretched over the row (`after:inset-0`) and the tag links sit above it (`z-10`), so don't nest anchors.
- `home.html` (hero text from `params.hero_title`, intro from the `about` page's `description`, about body, latest 3 posts), `blog/list.html`, `blog/single.html` (collapsible TOC above the post body; no sticky TOC column, to match the Index template), `section.html`, `term.html`, `taxonomy.html`, `page.html`. Sections with no dedicated layout (e.g. a future `projects/`) fall back to `section.html`.
- `_markup/` render hooks: heading (hover `#` anchor), blockquote (GitHub-style alerts, captioned and plain quotes), codeblock (Mermaid + highlighted code, always on a dark ground), image, table.
- `_partials/head/` — `css.html` (theme-init script, Tailwind CDN, token mapping, fonts), `js.html` (Swup; its `containers` are `#swup` and `#site-nav`), `katex.html`, `mermaid.html`, `opengraph.html`.

## Styling & theming

- Colors are CSS custom properties (`--c-canvas`, `--c-panel`, `--c-shell`, `--c-line`, `--c-ink`, `--c-muted`, `--c-accent*`, `--c-warn`, `--c-ok`) defined in `assets/css/theme.css` (processed via Hugo Pipes in `head/css.html`) for dark (default), `[data-theme="light"]`, and the system preference when no `data-theme` is set. `head/css.html` maps them into Tailwind with `@theme inline`, so templates use `bg-panel`, `text-muted`, `border-line`, `text-accent`, `bg-warn/10`, etc. Use these tokens, not raw grays or hex colors. Don't use Tailwind's `dark:` variant: it follows only `prefers-color-scheme`, not the toggle.
- Tailwind runs in the browser from the CDN, so classes must appear as complete strings in the templates (no string-built class names).
- Theme choice is `localStorage.theme` = `light` | `dark` | absent (system). An inline script at the top of `head/css.html` applies it before first paint. The toggle in `theme-toggle.html` sets/removes `data-theme` on `<html>`, persists the choice, and dispatches a `theme-changed` event (Mermaid re-renders in response and reads the effective theme, including system).
- Type: Geist for body, Geist Mono for display/labels/metadata (`font-display`, `font-mono`, `.section-label`, `.meta-mono`, `.display-hero`). Rendered Markdown sits in `.md-content`, whose list/paragraph/code/TOC styles are in `theme.css` because Tailwind's preflight strips browser defaults.
- Page transitions use Swup (`head/js.html`); the content container carries `swup-transition-fade`. Scripts inside swapped containers don't re-run, so keep behavior in once-run scripts with delegated listeners.

## Content authoring notes

- Post front matter fields in use: `title`, `date`, `author`, `tags`, `description` (also feeds Open Graph/Twitter meta via `head/opengraph.html`, falling back to `.Summary` when absent), and optionally `image` (relative to the page bundle) or `site.Params.og_image`.
- "Related posts" (`article/related.html`) are computed by intersecting `.Params.tags` against other pages under the same tag taxonomy — there's no manual linking.
- Mermaid diagrams work via a fenced code block with the `mermaid` language tag (handled in `_markup/render-codeblock.html`); DBML/ER-diagram support was discussed in `content/br/construcao/development-plan.md` but was never actually implemented — don't assume a DBML render hook exists.
- `content/br/construcao/*` are planning documents (`build.render/list: never`), not published pages — they record the original spec and phased plan for how this site was built, useful for context on *why* something looks the way it does.
