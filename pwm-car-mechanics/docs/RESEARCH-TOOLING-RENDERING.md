# Research: Build Tooling & Rendering Model for the PWM Website

**Date:** 2026-08-20
**Resolves:** [TICKET-PWM-002](../.wayfinder/tickets/TICKET-PWM-002-tooling-rendering-research.md) — bundler choice (Vite vs Rspack vs others) and rendering model (SPA vs SSR vs SSG).
**Method:** All claims below were checked against primary sources (official docs, release pages, MDN) in August 2026. Version numbers are what we observed live, not from memory.

---

## TL;DR Recommendation

> **Bundler: Vite (v8.x). Rendering model: Static Site Generation (SSG).**

Details, evidence, and rejected alternatives below. The concrete stack that implements this (plain Vite multi-page setup vs a framework like Astro) is decided by **PWM-005**, not here.

---

## What this repo already has (verified)

| Asset | Facts |
| --- | --- |
| `lessons/` | 8 standalone hand-written HTML pages (`0001`–`0008`) with inline vanilla-JS widgets and one shared stylesheet (`assets/style.css`). No build tooling today. |
| `pwm-presentation/` | Slidev deck pinned to `@slidev/cli ^52.18.0` + `vue ^3.5.33`, with 8 custom Vue components (`PwmSimulator.vue`, `QuizWidget.vue`, …). |
| Deploy configs | `netlify.toml` (publish `dist`, Node 24) and `vercel.json` (rewrite all → `/index.html`) — both live **inside `pwm-presentation/`** and are **SPA-scoped deck configs** (the catch-all rewrite serves the deck's single document). Correct for the deck; **not reusable** for a multi-page website. |

Two constraints follow directly:

1. Whatever we pick must emit **plain static files** — there is no server, database, or user accounts anywhere in the mission ([MISSION.md](../MISSION.md)).
2. We already have an investment in **Slidev**, which runs on **Vite**: Slidev's documented project structure includes a `vite.config.ts` extension point and its docs have a dedicated "Configure Vite and Plugins" section ([sli.dev/custom/directory-structure](https://sli.dev/custom/directory-structure)).

---

## Definitions (with sources)

- **SPA (Single-Page Application):** loads exactly one web document, then rewrites page content via JavaScript APIs like Fetch. Tradeoffs include SEO handling and reimplementing browser navigation/state management ([MDN Glossary](https://developer.mozilla.org/en-US/docs/Glossary/SPA)).
- **SSG (Static Site Generation):** all pages are generated ahead of time as plain HTML/CSS/JS files; every visitor gets identical content served fast from a CDN, with no server-side logic at request time ([MDN Glossary](https://developer.mozilla.org/en-US/docs/Glossary/SSG)).
- **SSR (Server-Side Rendering):** HTML is rendered per-request on a server. The sharpest framing we found: *"The only difference between SSG and SSR is when the HTML is rendered: SSG at build-time, SSR at request-time"* ([Vike — Pre-rendering](https://vite-plugin-ssr.com/pre-rendering)).

For a site of static lessons with small interactive widgets, the question is not *whether* the HTML can be pre-built (it obviously can) but whether anything forces us into per-request rendering. Nothing does — see reasoning below.

---

## Bundler comparison

| Tool | Current version (observed) | Dev experience | Production story | Ecosystem fit for us | Verdict |
| --- | --- | --- | --- | --- | --- |
| **Vite** | **v8.2.2** ([vite.dev](https://vite.dev)) | Instant ESM dev server | Since **Vite 8.0 (2026-03-12)** ships **Rolldown**, a Rust-based Rollup-compatible bundler, as the single unified engine — replacing the old esbuild+Rollup split, with full Rollup/Vite plugin compatibility ([announcement](https://vite.dev/blog/announcing-vite8)) | **What Slidev, Astro, Nuxt and SvelteKit are built on** — those framework teams tested `rolldown-vite` before the Vite 8 release ([announcement](https://vite.dev/blog/announcing-vite8)) | ✅ **Winner** |
| Rspack (+ Rsbuild) | Rspack **v2.1.10** stable (2026-08-13); v2.2.0-beta.1 pre-release ([releases](https://github.com/web-infra-dev/rspack/releases)) | Fast Rust bundler, webpack-compatible API | Solid; Rsbuild is its framework-agnostic config layer ([rsbuild.rs](https://rsbuild.rs/guide/start/)) | Its headline advantages — drop-in **webpack plugin compatibility** and **Module Federation** — target existing webpack apps ([Rsbuild comparisons](https://rsbuild.rs/guide/start/)). This repo has zero webpack legacy | Rejected |
| esbuild alone | Active | Very fast, minimal | No JS hot-reloading by design — *"outside of esbuild's scope"*; live-reload requires DIY watch mode plus hand-written client JS ([docs](https://esbuild.github.io/api/#live-reload)); its serve mode is dev-only ([docs](https://esbuild.github.io/api/#serve)) | We'd rebuild what Vite gives for free | Rejected |
| Parcel | Active | Zero-config by design ([parceljs.org](https://parceljs.org)) | Fine | Not what Slidev or the wider Vite-based ecosystem builds on; offers nothing extra for plain static output | Rejected |
| Rolldown standalone | Active ([rolldown.rs](https://rolldown.rs)) | Low-level library | Now primarily consumed *through* Vite 8 rather than used directly ([announcement](https://vite.dev/blog/announcing-vite8)) | Using it directly adds work with no benefit here | Rejected |

## Rendering model comparison

| Model | What it means | Fit for this repo | Verdict |
| --- | --- | --- | --- |
| **SSG** | Pages pre-built to static HTML/CSS/JS; identical content for every visitor, served from CDN ([MDN](https://developer.mozilla.org/en-US/docs/Glossary/SSG)) | Lessons are static content; widgets are self-contained vanilla JS needing no server data. `vite build` emits exactly these static assets, hostable anywhere ([Vite deploy guide](https://vite.dev/guide/build)) | ✅ **Winner** |
| Multi-page static (MPA flavor of SSG) | Multiple `.html` entry points built in one pass ([Vite MPA guide](https://vite.dev/guide/build#multi-page-app)) | Matches our existing 8 hand-written HTML lessons almost 1:1 | ✅ Recommended implementation shape |
| SPA | One document + client-side routing ([MDN](https://developer.mozilla.org/en-US/docs/Glossary/SPA)) | Adds SEO/navigation complexity for zero gain on static content; also requires the SPA rewrites our deck needed, whereas multi-page static HTML works on any host without them | Rejected |
| SSR | HTML rendered per request ([Vike](https://vite-plugin-ssr.com/pre-rendering)) | Needs a running server; we have no dynamic data to render | Rejected |

---

## Why Vite + SSG (reasoning)

1. **Output requirements are pure static files.** `vite build` produces static assets suitable for any static host ([vite.dev/guide/build](https://vite.dev/guide/build)), which is exactly what our existing `netlify.toml` / `vercel.json` already expect. SSG additionally means every learner gets identical files straight from a CDN — fast, secure, trivially cacheable ([MDN SSG](https://developer.mozilla.org/en-US/docs/Glossary/SSG)).
2. **Our interactivity doesn't need a server.** Every widget (PWM simulator, quiz, counters) is self-contained client-side JS operating on local state. SSR exists to render dynamic, per-request data ([Vike](https://vite-plugin-ssr.com/pre-rendering)); we have none. SPA routing would only add the SEO/state-management costs MDN warns about ([MDN SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA)).
3. **Toolchain alignment with our existing investment.** Slidev is built on Vite (its config surface *is* Vite config, [sli.dev/custom/directory-structure](https://sli.dev/custom/directory-structure)). Choosing Vite for the website means one bundler mental model across deck and site — significant for beginner maintainers.
4. **Beginner-friendliness.** A Vite multi-page app needs one config file listing HTML entries ([guide](https://vite.dev/guide/build#multi-page-app)); lessons can remain the plain HTML/CSS/JS they already understand. No framework runtime, no new component syntax required.
5. **Low deprecation risk.** Vite is the foundation under Slidev, Astro, Nuxt, and SvelteKit ([Vite 8 announcement](https://vite.dev/blog/announcing-vite8)), and Vite 8's Rolldown migration preserved Rollup-plugin compatibility — the ecosystem consolidated *on* it rather than away from it.

### Implementation note (for PWM-005, not a decision here)

Both of these satisfy "Vite + SSG":

- **Plain Vite MPA** — smallest concept count; keeps raw HTML lessons; best default given today's repo.
- **Astro** (currently **astro@7.2.4**, with `@astrojs/react@6.0.4`; [releases](https://github.com/withastro/astro/releases)) — an SSG framework *on top of* Vite whose islands architecture ships zero JS by default and hydrates individual components with `client:*` directives, including mixing React and Vue islands on the same page ([Astro Islands](https://docs.astro.build/en/concepts/islands/)). It also brings Markdown/content-collections authoring ([docs](https://docs.astro.build/en/guides/content-collections/)). Worth serious consideration **if** the lesson-format decision lands on Markdown.

---

## Rejected alternatives (one-liners)

- **Rspack/Rsbuild** — wins are webpack-compat and Module Federation ([rsbuild.rs](https://rsbuild.rs/guide/start/)); irrelevant for a greenfield site with no webpack history, and it's not what Slidev builds on.
- **Rspress** — Rspack-family static *React/MDX* generator ([rspack.rs ecosystem](https://rspack.rs/guide/start/ecosystem)); commits us to React before PWM-003 answers the component-sharing question.
- **esbuild alone** — explicitly no JS hot-reloading, DIY live-reload wiring ([esbuild docs](https://esbuild.github.io/api/#live-reload)).
- **Parcel** — capable zero-config bundler ([parceljs.org](https://parceljs.org)) but no ecosystem pull for us over Vite.
- **Next.js** — static export works but carries a long list of unsupported features (rewrites, redirects, ISR, server actions, default image optimization…) and App-Router complexity designed for dynamic apps ([official guide](https://nextjs.org/docs/app/guides/static-exports)).
- **Vike** — powerful "do-one-thing-do-it-well" Vite plugin ([vike.dev/add](https://vike.dev/add)) but explicitly more DIY boilerplate than beginners need.
- **Nuxt / SvelteKit** — SSR-first meta-frameworks tied to Vue/Svelte respectively ([nuxt.com](https://nuxt.com), [SvelteKit intro](https://svelte.dev/docs/kit/introduction)); overkill for static lessons.
- **SPA as rendering model** — extra complexity, no payoff for static content ([MDN SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA)).

---

## Open questions handed to other tickets

1. **→ PWM-003 (shared components / React?):** Slidev slides consume **Vue** components natively ([sli.dev/guide/component](https://sli.dev/guide/component)); React inside Slidev exists only via the third-party community addon [`slidev-addon-react`](https://github.com/Ygilany/slidev-addon-react). If shared widgets must be React, the website side is easy (any Vite setup, or Astro React islands), but the deck side needs that addon or a rewrite decision.
2. **→ PWM-004 (hosting):** recommendation assumes generic static hosting. The existing Netlify/Vercel configs are deck-scoped SPA artifacts and will **not** be reused: a multi-page static site needs its own config — publish the build output, zero redirects; any static file server is sufficient, PWM-004 picks the provider.
3. **Lesson format:** raw HTML (favors plain Vite MPA) vs Markdown/MDX (favors Astro content collections) — interacts directly with the implementation note above.
4. **i18n:** lessons are English while the presentation is Spanish; bilingual strategy unresolved and affects page structure regardless of tooling.

---

*Sources checked 2026-08-20 against official documentation and release pages; versions observed live on those pages.*
