# NalvoFilms – Frontend Architecture Showcase

> **Note:** This document is a technical showcase of selected patterns and solutions used in the NalvoFilms streaming platform. Source code is intentionally incomplete — this is not a deployable template.

Live: **[nalvofilms.com](https://nalvofilms.com)**  
Stack: Node.js · Express · MariaDB · Vanilla JS (ES12+) · Custom CSS Design System

---

## Table of Contents

- [Design System](#design-system)
- [Hero Section – Dynamic Backdrop](#hero-section--dynamic-backdrop)
- [Infinite Carousel – rAF Optimized](#infinite-carousel--raf-optimized)
- [Search – Debounced + XSS-safe](#search--debounced--xss-safe)
- [Card Microanimations](#card-microanimations)
- [Schema.org SSR](#schemaorg-ssr)
- [Real Analytics Module](#real-analytics-module)
- [Security Patterns](#security-patterns)

---

## Design System

Custom CSS design tokens — no Tailwind, no Bootstrap, pure CSS variables.

```css
:root {
  --nf-bg: #080a10;
  --nf-accent: #e50914;
  --nf-accent-glow: rgba(229, 9, 20, 0.5);
  --nf-text: #FFFFFF;
  --nf-text-muted: #A0AEC0;
  --nf-border: rgba(255, 255, 255, 0.08);
  --nf-radius: 16px;
  --nf-shadow-hover: 0 20px 50px rgba(0,0,0,0.55), 0 0 18px rgba(229,9,20,0.35);
  --nf-font: 'Inter', 'Lexend', system-ui, sans-serif;
  --transition-fast: 0.15s ease;
  --transition-base: 0.3s ease;
}
```

Font fallback with `size-adjust` to eliminate CLS (Cumulative Layout Shift) during web font swap:

```css
@font-face {
  font-family: 'Inter Fallback';
  src: local('Segoe UI'), local('system-ui'), local('Arial');
  size-adjust: 107%;
  ascent-override: 90%;
  descent-override: 22%;
  line-gap-override: 0%;
}
```

---

## Hero Section – Dynamic Backdrop

The hero rotates through featured titles every 8 seconds. Backdrop images are fetched from TMDB, preloaded, and crossfaded. The active item drives the background blur + gradient overlay.

```js
// Simplified excerpt — actual implementation omitted
const CONFIG = Object.freeze({
  HERO_ROTATION_INTERVAL: 8000,
  HERO_FIXED_IDS: Object.freeze([
    { id: 1405,   type: 'tv' },
    { id: 46952,  type: 'tv' },
    { id: 2288,   type: 'tv' },
    { id: 222766, type: 'tv' }
  ])
});

// State is kept in a single object — no framework needed
const state = {
  heroMovieList: [],
  heroCurrentIndex: 0,
  heroRotationInterval: null,
  heroLastDisplayedId: null,
};
```

Backdrop crossfade via CSS — no JS animation loop:

```css
.hero-backdrop {
  position: absolute;
  inset: 0;
  background-size: cover;
  background-position: center top;
  transition: opacity 0.9s cubic-bezier(0.4, 0, 0.2, 1);
  will-change: opacity;
}

.hero-backdrop.active  { opacity: 1; }
.hero-backdrop.leaving { opacity: 0; }

/* Cinematic gradient overlay */
.hero-streaming-wrap::after {
  content: '';
  position: absolute;
  inset: 0;
  background:
    linear-gradient(to right,  rgba(8,10,16,0.92) 0%, rgba(8,10,16,0.4) 60%, transparent 100%),
    linear-gradient(to top,    rgba(8,10,16,1)    0%, transparent 50%);
  pointer-events: none;
}
```

---

## Infinite Carousel – rAF Optimized

CSS `animation` based infinite scroll carousel. JS only sets the `--carousel-duration` custom property based on actual content width — no `setInterval`, no scroll events.

**The key insight:** batch all DOM reads in one `requestAnimationFrame`, then all DOM writes in the next — eliminates forced reflow.

```js
function syncCarouselSpeeds() {
  const tracks = document.querySelectorAll('.carousel-track');

  // OPTIMIZED: Batch all DOM reads first, then all DOM writes
  requestAnimationFrame(() => {
    // Phase 1: Read all measurements (no forced reflow)
    const measurements = Array.from(tracks).map(el => ({
      element: el,
      scrollWidth: el.scrollWidth
    }));

    // Phase 2: Write all styles (separate frame)
    requestAnimationFrame(() => {
      measurements.forEach(({ element, scrollWidth }) => {
        const duration = scrollWidth / CONFIG.CAROUSEL_SPEED_PX_PER_SEC;
        element.style.setProperty('--carousel-duration', `${duration}s`);
      });
    });
  });
}
```

```css
.carousel-track {
  display: flex;
  gap: 12px;
  animation: carouselScroll var(--carousel-duration, 120s) linear infinite;
  will-change: transform;
}

@keyframes carouselScroll {
  from { transform: translateX(0); }
  to   { transform: translateX(-50%); }
}

/* Pause on hover */
.carousel-wrap:hover .carousel-track {
  animation-play-state: paused;
}
```

---

## Search – Debounced + XSS-safe

Search hits the local `/api/content` endpoint (not TMDB directly), then enriches results with TMDB metadata. Input is debounced at 280ms and sanitized before any DOM insertion.

```js
/**
 * Sanitize HTML to prevent XSS — no library needed
 */
const sanitizeHTML = (str) => {
  if (typeof str !== 'string') return '';
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
};

/**
 * Escape HTML attributes
 */
const escapeAttr = (str) => {
  if (typeof str !== 'string') return '';
  return str.replace(/"/g, '&quot;').replace(/'/g, '&#39;');
};

/**
 * Generic debounce
 */
const debounce = (func, wait) => {
  let timeout;
  return function(...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, args), wait);
  };
};

// Usage
const handleSearchInput = debounce(async function(dropdown) {
  const query = sanitizeHTML(document.querySelector('#search-input').value.trim());
  if (query.length < 2) return;
  // ... fetch + render
}, 280);
```

---

## Card Microanimations

Netflix/HBO-style card hover: scale + lift + image zoom + overlay fade + title slide-up. All driven by CSS transitions, zero JS.

```css
.card-modern {
  transition: all 0.4s cubic-bezier(0.22, 1, 0.36, 1);
  transform-origin: center center;
  will-change: transform;
}

.card-modern:hover {
  transform: scale(1.08) translateY(-8px);
  z-index: 100;
  box-shadow:
    0 24px 48px rgba(0, 0, 0, 0.8),
    0 0 0 2px rgba(229, 9, 20, 0.3);
}

/* Image zoom */
.card-modern__image img {
  transition: transform 0.6s cubic-bezier(0.22, 1, 0.36, 1), filter 0.4s ease;
}
.card-modern:hover .card-modern__image img {
  transform: scale(1.1);
  filter: brightness(1.1);
}

/* Title slide-up on hover */
.card-modern__title {
  transform: translateY(10px);
  opacity: 0;
  transition: all 0.4s cubic-bezier(0.22, 1, 0.36, 1) 0.1s;
}
.card-modern:hover .card-modern__title {
  transform: translateY(0);
  opacity: 1;
}
```

---

## Schema.org SSR

Structured data (Movie / TVSeries / BreadcrumbList) is injected server-side into the HTML `<head>` — not generated client-side. This ensures Googlebot sees it on first crawl without JS execution.

```js
// modules/schema-ssr.js (excerpt)
function generateBreadcrumbSchema(contentType, tmdbId, title) {
  return {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    'itemListElement': [
      { '@type': 'ListItem', position: 1, name: 'Főoldal',   item: 'https://nalvofilms.com/' },
      { '@type': 'ListItem', position: 2, name: contentType === 'tv' ? 'Sorozatok' : 'Filmek' },
      ...(title ? [{ '@type': 'ListItem', position: 3, name: title }] : [])
    ]
  };
}
```

Injected in the Express route:

```js
// server.js (simplified)
app.get('/film/:id', async (req, res) => {
  const schema = schemaSSR.generateMovieSchema(tmdbData, 'movie');
  const html = await fs.readFile('details1.html', 'utf8');
  const injected = html.replace(
    '<!-- SCHEMA_PLACEHOLDER -->',
    `<script type="application/ld+json">${JSON.stringify(schema)}</script>`
  );
  res.send(injected);
});
```

---

## Real Analytics Module

Custom analytics — no Google Analytics, no Plausible. Tracks page views, session duration, and device type. Filters out known bot IPs and admin IPs via a database-backed exclusion list with in-memory cache.

```js
// modules/real-analytics.js (excerpt)
let excludedIpsCache = new Set();
let lastCacheUpdate  = 0;
const CACHE_TTL      = 5 * 60 * 1000; // 5 min

async function isExcludedIp(ip, pool) {
  if (Date.now() - lastCacheUpdate > CACHE_TTL) {
    const rows = await pool.query('SELECT ip_address FROM excluded_ips');
    excludedIpsCache = new Set(rows.map(r => r.ip_address));
    lastCacheUpdate  = Date.now();
  }
  return excludedIpsCache.has(ip);
}
```

GeoIP enrichment via `geoip-lite` — gracefully disabled if not installed:

```js
let geoip;
try {
  geoip = require('geoip-lite');
} catch (e) {
  console.warn('geoip-lite not installed — geolocation disabled');
  geoip = null;
}
```

---

## Security Patterns

### Rate limiting — no library, pure Map

```js
// server.js (excerpt)
const rateLimitStore = new Map(); // ip -> { count, resetTime }

function rateLimit(maxRequests = 100) {
  return (req, res, next) => {
    const ip  = getClientIp(req);
    const now = Date.now();
    const entry = rateLimitStore.get(ip);

    if (!entry || now > entry.resetTime) {
      rateLimitStore.set(ip, { count: 1, resetTime: now + 15 * 60 * 1000 });
      return next();
    }
    if (entry.count >= maxRequests) {
      return res.status(429).json({ error: 'too_many_requests' });
    }
    entry.count++;
    next();
  };
}
```

### CSRF token — crypto-based, IP-bound

```js
function generateCsrfToken(req) {
  const token = crypto.randomBytes(32).toString('hex');
  csrfTokens.set(token, {
    ip:      getClientIp(req),
    expires: Date.now() + 2 * 60 * 60 * 1000 // 2h
  });
  return token;
}

function validateCsrfToken(req, token) {
  const entry = csrfTokens.get(token);
  if (!entry || Date.now() > entry.expires) return false;
  return entry.ip === getClientIp(req); // IP-bound
}
```

### Input sanitization — no library

```js
function sanitizeInput(str, maxLength = 10000) {
  if (typeof str !== 'string') return '';
  return str
    .replace(/[<>]/g, '')  // strip < >
    .trim()
    .slice(0, maxLength);
}
```

---

## What's NOT here

The following are intentionally omitted from this showcase:

- Database schema and queries
- Authentication / session logic
- Video player source extraction
- `.env` configuration
- Admin panel
- Full HTML templates

---

*Built solo. Self-hosted on Ubuntu VPS behind Cloudflare + OpenResty.*  
*No frameworks were harmed in the making of this project.*
