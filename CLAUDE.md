# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Astro 5 blog (Cloudflare "astro-blog-starter-template", based on Bear Blog) deployed to Cloudflare Workers via `@astrojs/cloudflare`. Requires Node >=22. No test or lint tooling is configured.

## Commands

- `npm run dev` — dev server at localhost:4321
- `npm run build` — build to `./dist/` (worker at `dist/_worker.js/index.js`, static assets served via the `ASSETS` binding)
- `npm run preview` — build, then `wrangler dev`
- `npm run check` — full validation: `astro build && tsc && wrangler deploy --dry-run` (the closest thing to a test suite)
- `npm run deploy` — `wrangler deploy` (run `build` first)
- `npm run cf-typegen` — regenerate `worker-configuration.d.ts` after changing `wrangler.json` bindings

## Architecture

- **Content collection**: `src/content.config.ts` defines the `blog` collection (glob loader over `src/content/blog/**/*.{md,mdx}`). Frontmatter schema: `title`, `description`, `pubDate` (required), `updatedDate`, `heroImage` (optional; a string path, e.g. files in `public/`). Posts are consumed by `src/pages/blog/index.astro`, `src/pages/blog/[...slug].astro` (renders via `src/layouts/BlogPost.astro`), and `src/pages/rss.xml.js`.
- **Site-wide constants**: `SITE_TITLE` / `SITE_DESCRIPTION` in `src/consts.ts`, used by `BaseHead`, header, index, and RSS.
- **Site URL**: `astro.config.mjs` still has the placeholder `site: "https://example.com"`; canonical URLs, sitemap, and RSS links derive from it, so update it for the real domain.
- **Cloudflare**: `wrangler.json` points `main` at the built worker and serves `./dist` as assets; observability and source-map upload are enabled. `platformProxy` is on so `astro dev` emulates Workers bindings. `public/.assetsignore` controls which files Wrangler skips when uploading assets.
- **Styling**: single global stylesheet `src/styles/global.css` (no CSS framework); fonts are in `public/fonts/`.
