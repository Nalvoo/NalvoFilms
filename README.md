# NalvoFilms

Hungarian streaming platform built without a UI framework. Custom auth, analytics, and SSR.

**Live:** [nalvofilms.com](https://nalvofilms.com) &nbsp;|&nbsp; Self-hosted on Ubuntu VPS · Cloudflare · OpenResty

---

## What it is

NalvoFilms is a self-hosted streaming platform for Hungarian audiences. Users browse a TMDB-backed catalogue, watch trailers, manage a watchlist, and submit content requests. The platform has its own auth system, user profiles, an admin panel, and custom analytics. Nothing is outsourced to third-party services.

No React, no Vue, no Next.js. Vanilla JS, Express, MariaDB, and a custom CSS design system.

---

## Why no framework

A streaming platform doesn't need a virtual DOM. It needs fast pages, clean HTML for SEO, and a server that can inject structured data before the response lands in the browser.

The no-framework constraint was a deliberate call from the start, not a retrofit. The result loads faster than most framework-based equivalents, has no build pipeline, no hydration mismatches, and no 300kb JS bundle to parse before anything renders.

---

## What's in it

- **Catalogue** backed by TMDB with SSR detail pages. Schema.org structured data is injected server-side so Googlebot sees it without JS execution.
- **Hero section** rotating featured titles with backdrop crossfade. CSS transitions only, no JS animation loop.
- **Infinite carousel** driven by CSS animation. JS only sets the speed based on measured content width.
- **Search** debounced at 280ms, hitting a local API endpoint rather than TMDB directly. XSS-safe without a library.
- **Auth system** covering registration, login, email verification, password reset, and session management.
- **User profiles** with watchlist and content request submissions.
- **Admin panel** for content management, analytics, and user management.
- **Custom analytics** with no Google Analytics and no Plausible. Tracks page views, session duration, and device type. Bot exclusion via IP allowlist with in-memory TTL cache.
- **Rate limiting and CSRF** written from scratch without express-rate-limit or csurf.
- **PWA** manifest with offline support.

---

## Stack

Node.js · Express · MariaDB · Vanilla JS (ES12+) · Custom CSS · OpenResty · Cloudflare

---

## Selected patterns

**rAF-batched carousel.** JS reads all DOM measurements in one `requestAnimationFrame` and writes all styles in the next. Eliminates forced reflow across multiple carousel tracks.

**Schema.org SSR.** Structured data is injected into `<head>` in the Express route before the response is sent. No client-side generation, no JS dependency for Googlebot.

**XSS sanitization without a library.** Input is assigned to `div.textContent` then read back as `innerHTML`. The browser does the escaping without DOMPurify or regex.

**Font fallback with size-adjust.** A `@font-face` fallback stack with `size-adjust`, `ascent-override`, and `descent-override` eliminates CLS during web font swap.

**Process-local analytics exclusion cache.** Bot and admin IPs are stored in DB and cached in-memory with a 5-minute TTL. The DB is not hit on every pageview.

---

## My role

I'm not primarily a developer. What I did:

- Defined the product, what it does, what it doesn't, and what gets built next
- Made every feature, UX, and architecture decision including the no-framework constraint
- Ran all deployment and ops: VPS setup, nginx, Cloudflare, domain, monitoring
- Handled SEO audits, performance optimization rounds, and accessibility fixes

Implementation was done with AI coding agents under my direction. I wrote the specs, reviewed outputs, caught regressions, and owned every production decision.

---

## What's not in this repo

This is a showcase of architecture and selected patterns. Auth logic, DB schema, and video extraction are intentionally omitted.

---

## Contact

Built by **Nalvo** · [nalvo.hu](https://nalvo.hu)
