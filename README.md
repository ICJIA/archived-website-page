# ICJIA — Archived Site Placeholder

![Preview of the ICJIA archived-site placeholder page rendered at desktop width: centered ICJIA wordmark above an elevated "This site has been archived" card with a contact section, followed by a centered link to icjia.illinois.gov and a centered copyright footer, all on a dark navy background.](assets/screenshot.png)

A drop-in static page for any ICJIA Netlify site that has been archived. Dark-mode, WCAG 2.1 AA verified, zero external runtime dependencies, locked-down security headers.

## Quick start

```sh
npm install
npm run dev          # boots Tailwind watcher + static server on :3000
```

Open <http://localhost:3000>.

> **Note:** local `serve` does not apply the Netlify response headers from `netlify.toml` / `_headers`. The rendered page is identical; to test headers locally, use `npx netlify dev` after installing the Netlify CLI.

## Deploy to Netlify

Point the target Netlify site at this repo. Netlify reads `netlify.toml` and runs the build:

- **Build command:** `npm install && npm run build:css`
- **Publish directory:** `.` (repo root)
- **Result:** `index.html`, generated `styles.css`, and `assets/` are served as static files; security headers are applied per `netlify.toml`.

No environment variables required. True drop-in.

## Stack

- Single static `index.html` — no framework, no client routing.
- **Tailwind CSS, compiled locally** (CLI builds to `styles.css` at deploy time). No CDN, no runtime JIT.
- One ~80-byte external script (`assets/year.js`) sets the copyright year via `textContent`.
- Self-hosted official ICJIA wordmark as vector SVG (`assets/icjia-logo.svg`, ~13 KB) — derived from ICJIA's master EPS via Ghostscript + `pdftocairo` (see "Updating the logo" below).
- System font stack — zero webfont network calls.

## Files

```
.
├── index.html              # the page
├── src/input.css           # Tailwind source (@tailwind directives + base layer rules)
├── styles.css              # built CSS (gitignored, produced by `npm run build:css`)
├── assets/
│   ├── icjia-logo.svg      # official ICJIA wordmark (vector)
│   └── year.js             # one-line dynamic copyright year script
├── tailwind.config.js      # Tailwind content paths
├── package.json            # tailwindcss / serve / concurrently devDeps
├── netlify.toml            # build command + response headers
├── _headers                # backup of header rules (alternate Netlify format)
└── README.md
```

## Editing

- **Copy:** edit the `<h1>` and lead paragraph in `index.html`.
- **Contact email:** `cja.info@icjia.illinois.gov` appears once in the contact card.
- **Colors:** dark palette lives in `src/input.css` (`#0b1220` page bg, `#60a5fa` accent, slate text scale). Re-run `npm run build:css` after edits, or use `npm run dev` (auto-rebuilds on save).
- **Logo:** drop a replacement at `assets/icjia-logo.svg`, or swap to a different format by editing the `<img src>` and `width`/`height` attributes in `index.html`. **EPS is not web-renderable** — see "Updating the logo" below for the conversion recipe.

### Updating the logo

The current logo was converted from `ICJIAwhite [Converted].eps` (ICJIA's master white-on-transparent wordmark) via Ghostscript + `pdftocairo`. To regenerate from a fresh EPS source:

```sh
# requires: brew install ghostscript poppler  (poppler provides pdftocairo)
gs -q -dNOPAUSE -dBATCH -sDEVICE=pdfwrite -dEPSCrop \
   -sOutputFile=/tmp/icjia.pdf "/path/to/ICJIAwhite.eps"
pdftocairo -svg /tmp/icjia.pdf assets/icjia-logo.svg
```

The output is a self-contained SVG (no external font deps, no remote refs). For a black-on-white deployment, repeat with `ICJIAblack.eps` and tweak the page palette accordingly.

## Audit results

Last audited **2026-05-12** against the locally served build.

| Audit | Tool | Result |
| --- | --- | --- |
| Lighthouse Accessibility | `lightcap` MCP | **100 / 100** |
| Lighthouse Performance | `lightcap` MCP | **100 / 100** |
| Lighthouse Best Practices | `lightcap` MCP | **100 / 100** |
| Lighthouse SEO | `lightcap` MCP | **100 / 100** |
| axe-core (WCAG 2.1 AA) | `axecap` MCP | **0 violations** |
| Contrast (WCAG 2.1 AA) | `contrastcap` MCP | **10 / 10 pass** |
| npm dependency vulnerabilities | `npm audit` | **0 vulnerabilities** |

axe-core flags one informational "needs review" item on a `span[aria-hidden="true"]` (the decorative `→` arrow). `aria-hidden` content is intentionally outside the accessibility tree, so this is not a real failure — axe-core just declines to auto-decide.

## Security posture

### Response headers (applied by Netlify)

```
Content-Security-Policy: default-src 'none'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'none'
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=(), interest-cohort=(), accelerometer=(), gyroscope=(), magnetometer=(), payment=(), usb=()
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Resource-Policy: same-origin
```

### Notable hardening

- **`default-src 'none'`** — nothing loads unless explicitly allowed by a more specific directive.
- **No `'unsafe-inline'`** anywhere. All scripts and styles are external, same-origin.
- **No third-party origins** in any CSP directive — page truly loads zero remote resources.
- **HSTS** is 2 years with `preload` (eligible for the HSTS preload list).
- **Cross-origin isolation** — COOP/COEP/CORP set to `same-origin` / `require-corp` / `same-origin`. The page operates in an isolated browsing context.
- **Permissions-Policy** denies all sensor, payment, and USB APIs. The page neither uses nor needs them.

### Verifying after deploy

```sh
URL=https://your-archived-site.netlify.app  # replace with the real deployed URL

# Inspect security headers
curl -sI "$URL" | grep -Ei '^(content-security-policy|strict-transport-security|x-frame-options|x-content-type-options|referrer-policy|permissions-policy|cross-origin-)'

# Confirm no third-party requests on page load
# (Open DevTools → Network → reload — every request should be same-origin or `data:`)
```

External validators:

- [Mozilla Observatory](https://observatory.mozilla.org/) — should grade **A+**.
- [HSTS Preload status](https://hstspreload.org/) — confirms preload eligibility.
- [securityheaders.com](https://securityheaders.com/) — should grade **A+**.

## Red / blue team review

Condensed adversarial analysis. Last reviewed **2026-05-12**.

### Threat model

Public, read-only static page on Netlify under an ICJIA-controlled domain. No user accounts, no input handling, no PII collected, no database. Plausible adversaries: opportunistic scanners, defacement attempts, automated XSS/injection probes, supply-chain attackers.

### Red team — attack surface

| # | Vector | Status |
| --- | --- | --- |
| 1 | Reflected / stored / DOM XSS | **Eliminated** — no user input, no query-string parsing, no `innerHTML` writes. The only JS sets `textContent`, which never parses HTML. |
| 2 | Clickjacking | **Blocked** — `X-Frame-Options: DENY` + CSP `frame-ancestors 'none'`. |
| 3 | MIME-type confusion | **Blocked** — `X-Content-Type-Options: nosniff`. |
| 4 | TLS downgrade / SSL strip | **Blocked** — HSTS 2y + `preload`. |
| 5 | Mixed-content downgrade | **N/A** — zero remote subresource loads. |
| 6 | CSP bypass via inline injection | **N/A** — `'unsafe-inline'` is absent from both `script-src` and `style-src`. |
| 7 | CSP bypass via third-party CDN | **N/A** — no third-party origins in any directive. |
| 8 | Runtime third-party JS compromise (e.g. CDN takeover) | **Eliminated** by migrating Tailwind from the Play CDN to a local build. |
| 9 | npm supply-chain (devDep compromise) | **Build-time only** — devDeps (`tailwindcss`, `serve`, `concurrently`) never ship to the published artifact. `npm audit` clean. `package-lock.json` pins resolved versions. |
| 10 | Image-based attacks (stego / oversize) on the logo | **Low** — logo is a self-hosted 6.7 KB PNG over a same-origin path. No user uploads. |
| 11 | CSRF / state-changing requests | **N/A** — no state-changing endpoints. |
| 12 | Open redirect | **N/A** — no redirect endpoints. |
| 13 | Path traversal | **N/A** — Netlify serves static files from a fixed publish directory. |
| 14 | Cookie / session hijacking | **N/A** — no cookies, no sessions. |
| 15 | Cache poisoning | **Low** — Netlify edge cache; static assets only; no `Vary` games. |
| 16 | Phishing via lookalike domain | **Out of scope**; HSTS preload on the real domain partially mitigates. |
| 17 | Email spoofing of `cja.info@icjia.illinois.gov` | **Out of scope** — depends on SPF/DKIM/DMARC on `illinois.gov`. |
| 18 | DNS / BGP hijacking | **Out of scope** (registrar/provider control); partially mitigated by HSTS preload. |
| 19 | Netlify platform compromise | **Trust dependency**; out of scope. |
| 20 | Browser zero-day | **Out of scope**. |

### Blue team — controls in place

- **Network:** HTTPS-only via Netlify; HSTS `max-age=63072000; includeSubDomains; preload`.
- **CSP:** `default-src 'none'`, no `'unsafe-inline'` or `'unsafe-eval'`, no third-party origins. Removes whole vector classes (XSS, third-party drift) on this page.
- **Framing / embedding:** `X-Frame-Options: DENY` + CSP `frame-ancestors 'none'` + `Cross-Origin-Resource-Policy: same-origin`.
- **Cross-origin isolation:** COOP `same-origin`, COEP `require-corp`. Page is cross-origin isolated.
- **MIME safety:** `X-Content-Type-Options: nosniff`.
- **Referrer minimization:** `Referrer-Policy: strict-origin-when-cross-origin`.
- **Capability denial:** `Permissions-Policy` denies camera, mic, geolocation, sensors, payment, USB, FLoC.
- **Code hygiene:** no inline `<script>`, no inline `<style>`, no inline event handlers, no `eval`, no `innerHTML`. The single DOM write is `element.textContent = …`, which is safe by definition.
- **Supply chain:** `package-lock.json` pins resolved versions; `npm audit` shows 0 vulnerabilities; devDeps never reach production.
- **Assets:** logo and script are self-hosted; favicon is an inline data URI; no third-party hotlinking.
- **No PII collection:** no forms, no analytics, no cookies.

### Residual risks

- **Trust in Netlify** — build pipeline, edge, TLS termination. Unavoidable for Netlify-hosted properties.
- **Trust in the npm registry** and the `tailwindcss` / `serve` / `concurrently` maintainer chains. Limited blast radius (build-time only), but worth periodic re-audits (`npm audit`, Dependabot, or Renovate).
- **`Cross-Origin-Embedder-Policy: require-corp` is strict.** Future external resources (e.g. a webfont) will require either dropping COEP or ensuring the resource serves a permissive CORP/CORS header. This is intentional friction.
- **DNS / domain control** sits with the ICJIA domain administrator. HSTS preload helps but is not a substitute for registrar 2FA and DNSSEC.

### Recommended ongoing maintenance

1. Re-run `npm audit` and refresh `package-lock.json` quarterly, or wire up Dependabot / Renovate.
2. After each Netlify deploy, validate headers with the `curl` snippet above and aim for an A+ on Mozilla Observatory.
3. If you add any external resource (font, image, analytics), update both the relevant CSP `*-src` directive **and** re-run the audits documented here.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for the version history.

## License

Released under the [MIT License](LICENSE). © 2026 Illinois Criminal Justice Information Authority.
