# Vite bundler with fully static output for the Website

Status: accepted (2026-08-20)

The Website's lessons are static content with self-contained client-side widgets; nothing requires per-request rendering, so every page is fully assembled before deploy and served identically to every visitor from a CDN. We adopt **Vite** as the bundler ahead of the site-architecture work (PWM-005) because that work — landing page, lessons index, shared navigation across ~10 pages, reusable widget code — creates the duplication and shared-code pain a bundler solves, and adopting first means building on the tool instead of migrating onto it mid-project. Terminology note: this decision is **static output, not SSG** — no tool generates HTML here; the word SSG is reserved for a possible future framework that does.

## Scope

This decision fixes the toolchain and rendering model only (PWM-002). Lesson format (raw HTML vs Markdown) and page architecture (plain Vite MPA vs a generator like Astro) are deliberately deferred to PWM-005; hosting provider is PWM-004's.

## Considered Options

- **Rspack/Rsbuild** — wins are webpack compatibility and Module Federation; irrelevant with zero webpack legacy.
- **esbuild alone** — no JS live-reload by design; we would rebuild what Vite gives for free.
- **Parcel** — capable, but no ecosystem pull over Vite, which is what Slidev already builds on.
- **Next.js / Nuxt / SvelteKit / Vike** — dynamic-app machinery or DIY boilerplate disproportionate to static lessons.
- **Astro** — not rejected, *deferred*: it satisfies this decision's invariant and remains a live PWM-005 option if lesson format lands on Markdown.
- **SPA rendering** — adds routing/SEO complexity for zero gain on static content.

## Consequences

- **Framework neutrality:** Vite is the intersection of every PWM-003 outcome — React via plugin, web components natively, Vue/Slidev unchanged — which is why this could be decided before PWM-003. Revisit trigger: PWM-003 identifies a requirement Vite cannot support.
- **Deploy story:** a multi-page static build needs no host routing infrastructure; any static file server suffices. The existing Netlify/Vercel configs are deck-scoped SPA artifacts (catch-all rewrite) and must not be reused for the Website.
- **Risk budget:** Vite 8's Rolldown engine is new (2026-03), but our exposure surface is minimal by construction — plain MPA, vanilla JS/CSS, no custom plugins. Failures surface at build/preview time, never silently; pin the exact version; downgrade to maintained Vite 7 is a one-line change.
- **Known non-goals:** Vite fixes none of the problems currently listed in the README (diagram alignment, missing index/navigation); those are PWM-005/PWM-006 work. Bilingual structure (English lessons, Spanish deck) constrains PWM-005, not tooling.
