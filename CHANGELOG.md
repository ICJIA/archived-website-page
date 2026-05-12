# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-05-12

Initial public release of the ICJIA archived-site placeholder.

### Added

- Single-page static site (`index.html`) declaring archived-but-not-deleted status, with a centered ICJIA wordmark, elevated content card (H1 + lead paragraph + contact section), centered outbound link to `icjia.illinois.gov`, and centered footer.
- Official ICJIA wordmark rendered as self-contained vector SVG (`assets/icjia-logo.svg`), converted from the master EPS source via Ghostscript + `pdftocairo`. Conversion recipe documented in `README.md`.
- Dark-mode visual system: `#0b1220` page background, `slate-800` elevated content card, `slate-100`/`slate-300`/`slate-400` text scale, `#60a5fa` (blue-400) accent, soft radial gradient via `body::before` pseudo-element.
- Local Tailwind CSS build pipeline (`tailwindcss` CLI). Source in `src/input.css`, compiled to `styles.css` (~9 KB minified) at deploy time. No CDN, no runtime JIT.
- Dynamic copyright year via `assets/year.js` (one-line `textContent` assignment with hardcoded fallback).
- Entrance fade-in animation on main content, automatically skipped under `prefers-reduced-motion`.
- Dev tooling: `npm run dev` runs Tailwind in watch mode alongside a static `serve` instance on `:3000` (via `concurrently`).
- Netlify build pipeline via `netlify.toml` — `npm install && npm run build:css`, publish `.`, no env vars required.
- Backup `_headers` file mirroring the headers defined in `netlify.toml`.

### Security

- Strict Content-Security-Policy: `default-src 'none'`; explicit allowlists per directive; no `'unsafe-inline'` for either `script-src` or `style-src`; no third-party origins anywhere.
- `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload` (2 years, eligible for HSTS preload list).
- Cross-origin isolation: `Cross-Origin-Opener-Policy: same-origin`, `Cross-Origin-Embedder-Policy: require-corp`, `Cross-Origin-Resource-Policy: same-origin`.
- `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`.
- `Permissions-Policy` denying camera, microphone, geolocation, FLoC, accelerometer, gyroscope, magnetometer, payment, and USB APIs.
- All assets self-hosted (logo, CSS, JS, favicon as inline data URI). Zero runtime remote dependencies.
- `npm audit` clean (0 vulnerabilities at release).
- Red/blue team analysis documented in `README.md`.

### Accessibility

- WCAG 2.1 AA verified.
- Lighthouse: Accessibility **100**, Performance **100**, Best Practices **100**, SEO **100** (mobile viewport).
- axe-core AA: **0 violations**.
- contrastcap WCAG AA: **10 / 10 pass**.
- Semantic landmarks (`<header>` / `<main>` / `<footer>`), single `<h1>`, visible focus rings, no color-only meaning.

[1.0.0]: https://github.com/ICJIA/archived-website-page/releases/tag/v1.0.0
