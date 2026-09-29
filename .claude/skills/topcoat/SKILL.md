---
name: topcoat
description: Evaluate and, if feasible, migrate Git Profile Age to the Topcoat Rust framework (tokio-rs/topcoat) while keeping GitHub Pages deployment. Use when the user asks to revisit the Topcoat rewrite.
---

# Rewrite Git Profile Age with Topcoat

Deferred on 2026-09-30. Topcoat (https://github.com/tokio-rs/topcoat) was
server-rendered only: it needed a running Tokio process, and "Static export"
and "Pre-rendering for static pages" were still roadmap items (latest release
at the time: v0.9.0, 2026-09-24). GitHub Pages serves static files only, so
the rewrite was not possible.

## Step 1: Verify that the blocker is gone

Do this before touching any code. Report the findings to the user and stop if
any check fails.

1. Fetch https://raw.githubusercontent.com/tokio-rs/topcoat/main/README.md and
   the latest release notes at https://github.com/tokio-rs/topcoat/releases.
2. Confirm that static export (or pre-rendering to plain HTML/JS/CSS with no
   server at runtime) is implemented and documented, not just listed in the
   roadmap.
3. Confirm the exported output works without any Topcoat server process:
   no WebSocket server push, no htmx/Datastar endpoints, no proxy.
4. Confirm that browser-side code can still do everything the current
   `index.html` does from the client: `fetch` to third-party Git APIs,
   `localStorage`, Web Share API, Clipboard API, `history.replaceState`,
   reading `?url=` on load. Check the `$(...)` runtime expression docs and
   whether `raw!` JavaScript is allowed in exported output.
5. Check the stability statement. If the README still says "Early-stage and
   experimental. Expect breaking changes.", tell the user and ask whether to
   proceed anyway.

## Step 2: Confirm the trade-offs with the user

The project's `CLAUDE.md` requires Vanilla JS, plain HTML/CSS, no frameworks,
no build system and a single `index.html`. A Topcoat rewrite changes all of
that. Before implementing, get explicit confirmation and then update
`CLAUDE.md` so the constraints describe the new architecture.

Non-negotiable properties that must survive the rewrite:

- Frontend-only at runtime: no project backend, no project-side logging.
- Git API requests go straight from the browser to the Git service.
- No analytics, tracking, cookies, API keys or secrets.
- Deployed to GitHub Pages from the `main` branch via
  `.github/workflows/` (add a Rust build step that produces the static
  export and uploads it with `actions/upload-pages-artifact`).
- All behaviour listed in `README.md` keeps working: service adapters, URL
  normalization, `?url=` auto-lookup, sharing, caching with storage
  fallback, loading and error states, accessibility, SEO metadata,
  `favicon.svg`, `404.html`, `robots.txt`, `sitemap.xml`,
  `site.webmanifest`, `.well-known/`, `og-image.png`.

## Step 3: Migration plan

1. Bootstrap a Topcoat project in the repository root (`topcoat new` if it
   exists), keeping the current `index.html` until the new build reaches
   parity.
2. Port the page structure and CSS into Topcoat views.
3. Keep the service-adapter registry and lookup logic as client-side code.
   Either port it to `$(...)` expressions where the subset allows, or keep it
   as a plain JavaScript asset served through the asset pipeline. Do not
   move API calls to the server.
4. Configure static export so the output directory contains the site root
   with `index.html`, `404.html` and all static files listed above.
5. Update the GitHub Pages workflow: install Rust, build, export, upload the
   export directory.
6. Test everything in the Definition of Done section of `CLAUDE.md` against
   the exported output served by a plain static file server.
7. Update `README.md`, `CONTRIBUTING.md` and `CLAUDE.md` to describe the real
   implementation, local development (Rust toolchain, `topcoat` CLI) and the
   build step.
8. Remove the old `index.html` only after the exported site passes all tests.

## If the blocker is still there

Tell the user which check failed, cite the Topcoat README or release notes,
and recommend staying on the current Vanilla JS implementation. Do not start
a partial migration.
