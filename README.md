# NalvoFilms

Hungarian streaming platform built without a UI framework. Custom auth, analytics, and SSR.

**Live:** [nalvofilms.com](https://nalvofilms.com) &nbsp;|&nbsp; Self-hosted on Ubuntu VPS · Cloudflare · OpenResty

---

## What it is

NalvoFilms is a self-hosted streaming platform for Hungarian audiences. Users browse a TMDB-backed catalogue, watch trailers, manage a watchlist, and submit content requests. The platform has its own auth system, user profiles, an admin panel, and custom analytics — nothing outsourced to third-party services.

No React, no Vue, no Next.js. Vanilla JS, Express, MariaDB, and a custom CSS design system.

---

## Why no framework

A streaming platform doesn't need a virtual DOM. It needs fast pages, clean HTML for SEO, and a server that can inject structured data before the response lands in the browser.

The no-framework constraint was a deliberate call from the start, not a retrofit. The result loads faster than most framework-based equivalents, has no build pipeline, no hydration mismatches, and no 300kb JS bundle to parse before anything renders.

---

## What's in it

- **Catalogue** backed by TMDB — SSR detail pages with Schema.org structured data injected server-side so Googlebot sees it without JS execution
- **Hero section** rotating featured titles with backdrop crossfade — CSS transitions, no JS animation loop
- **Infinite carousel** driven by CSS animation, JS only sets the speed based on measured content width
- **Search** debounced at 280ms, hits a local API endpoint (not TMDB directly), XSS-safe without a library
- **Auth system** — registration, login, email verification, password reset, session management
- **User profiles** with watchlist and content request submissions
- **Admin panel** — content management, analytics dashboard, user management
- **Custom analytics** — no Google Analytics, no Plausible. Page views, session duration, device type, bot exclusion via IP allowlist with in-memory TTL cache
- **Rate limiting and CSRF** — written from scratch, no express-rate-limit or csurf dependency
- **PWA** — installable, offline manifest

---

## Stack

Node.js · Express · MariaDB · Vanilla JS (ES12+) · Custom CSS · OpenResty · Cloudflare

---

## Selected patterns

**rAF-batched carousel.** JS reads all DOM measurements in one `requestAnimationFrame`, writes all styles in the next — eliminates forced reflow across multiple carousel tracks.

**Schema.org SSR.** Structured data (Movie / TVSeries / BreadcrumbList) is injected into `<head>` in the Express route before the response is sent. No client-side generation, no JS dependency for Googlebot.

**XSS sanitization without a library.** Input assigned to `div.textContent`, then read back as `innerHTML` — browser does the escaping, no DOMPurify, no regex.

**Font fallback with size-adjust.** `@font-face` fallback stack with `size-adjust`, `ascent-override`, `descent-override` to eliminate CLS during web font swap.

**Process-local analytics exclusion cache.** Bot and admin IPs stored in DB, cached in-memory with a 5-minute TTL — DB not hit on every pageview.

---

## My role

I'm not primarily a developer. What I did:

- Defined the product — what it does, what it doesn't, what gets built next
- Made every feature, UX, and architecture decision (including the no-framework constraint)
- Ran all deployment and ops: VPS setup, nginx, Cloudflare, domain, monitoring
- Handled SEO audits, performance optimization rounds, and accessibility fixes

Implementation was done with AI coding agents under my direction. I wrote the specs, reviewed outputs, caught regressions, and owned every production decision.

---

## What's not in this repo

This is a showcase of architecture and selected patterns. Auth logic, DB schema, and video extraction are intentionally omitted.

---

## Contact

Built by **Nalvo** · [nalvo.hu](https://nalvo.hu)
