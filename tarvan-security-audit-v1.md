# TARVAN Website — Security Audit Report

**Audit type:** Static site security review (HTML + inline CSS + inline JS)
**Date:** 2026-03-07
**Auditor:** Dr. Shanmugakumar / TARVAN Engineering
**Scope:** `dist/index.html` + `dist/assets/*` — the production deployment artifact
**Method:** Shanmuga Engineering Method — Phase 3 (Security-First), threat-model-driven

---

## Scope Definition

**What is being audited:** A single-page static website (`index.html`, ~74KB) with inline CSS, inline JS, and static image assets. No backend, no forms, no user input, no database, no authentication. Hosted as a static site (likely Cloudflare Pages, Netlify, or similar).

**Threat model (STRIDE for static sites):**

| Threat | Applicability | Reasoning |
|---|---|---|
| Spoofing | LOW | No authentication. But domain/DNS spoofing is an infrastructure concern. |
| Tampering | MEDIUM | If the hosting is compromised, the HTML can be modified. Supply chain risk via Google Fonts. |
| Repudiation | LOW | No user actions to repudiate. |
| Information Disclosure | MEDIUM | `.DS_Store` files, email exposure, source code comments. |
| Denial of Service | LOW | Static site behind CDN is inherently resilient. |
| Elevation of Privilege | N/A | No privilege model exists. |

**Primary risk surface:** This is a B2B corporate website. The threat is not "hack the app" — it's "damage the brand, harvest the founder's email, or inject malicious content via supply chain."

---

## Findings

### FINDING 1 — No Content Security Policy (CSP)
**Severity: HIGH**
**Category: Tampering / XSS Defense**

The page has zero CSP headers — no `<meta http-equiv="Content-Security-Policy">` tag, and no indication of server-side CSP headers.

**Risk:** Without CSP, if an attacker manages to inject content (e.g., via a compromised CDN, a hosting misconfiguration, or a browser extension), there is no browser-enforced boundary preventing inline script execution, external resource loading, or data exfiltration.

**Remediation — add this meta tag to `<head>`:**

```html
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self';
  script-src 'self' 'unsafe-inline';
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  font-src 'self' https://fonts.gstatic.com;
  img-src 'self' data:;
  connect-src 'self';
  frame-src 'none';
  object-src 'none';
  base-uri 'self';
  form-action 'none';
">
```

**Notes:**
- `'unsafe-inline'` is required because all CSS and JS are inline. This is a trade-off — inline code avoids extra HTTP requests for a single-page site, but it weakens CSP. Acceptable for a static brochure site with no user input.
- `frame-src 'none'` prevents the site from being iframed (clickjacking defense).
- `object-src 'none'` blocks Flash/Java plugins.
- `form-action 'none'` prevents form submission to external targets (defense-in-depth even though there are no forms).
- `base-uri 'self'` prevents `<base>` tag injection.

---

### FINDING 2 — No X-Frame-Options / Clickjacking Protection
**Severity: MEDIUM**
**Category: Tampering**

No `X-Frame-Options` header or meta tag. The site can be embedded in an `<iframe>` on a malicious page.

**Risk:** An attacker could embed tarvan.ai in a frame, overlay fake UI elements on top, and trick visitors into clicking malicious links while thinking they're interacting with TARVAN's site. For a B2B site with `mailto:` CTAs, this could be used for phishing.

**Remediation — add to `<head>`:**

```html
<meta http-equiv="X-Frame-Options" content="DENY">
```

This is also covered by the CSP `frame-ancestors 'none'` directive (add to the CSP above for belt-and-suspenders):

```
frame-ancestors 'none';
```

---

### FINDING 3 — No Referrer-Policy
**Severity: MEDIUM**
**Category: Information Disclosure**

No `Referrer-Policy` meta tag. When a visitor clicks an external link (LinkedIn, matic.one), the full referring URL is sent to the target site by default.

**Risk:** Leaks the exact page URL (including any query parameters that future analytics might add) to third-party sites. For a simple static page this is low-impact today, but it's a hygiene failure.

**Remediation — add to `<head>`:**

```html
<meta name="referrer" content="strict-origin-when-cross-origin">
```

This sends only the origin (`https://tarvan.ai`) to cross-origin destinations, not the full path.

---

### FINDING 4 — No Permissions-Policy
**Severity: LOW**
**Category: Defense in Depth**

No `Permissions-Policy` (formerly `Feature-Policy`) to restrict browser features.

**Risk:** If the site is ever compromised or iframed, an attacker could use browser APIs (camera, microphone, geolocation) through injected scripts.

**Remediation — add to `<head>`:**

```html
<meta http-equiv="Permissions-Policy" content="camera=(), microphone=(), geolocation=(), payment=()">
```

---

### FINDING 5 — Google Fonts Loaded Without SRI
**Severity: MEDIUM**
**Category: Supply Chain / Tampering**

```html
<link href="https://fonts.googleapis.com/css2?family=Orbitron:...&family=Saira:...&display=swap" rel="stylesheet">
```

This loads a CSS file from Google's servers with **no Subresource Integrity (SRI) hash**. Google Fonts dynamically generates CSS based on user-agent, making traditional SRI impossible for this endpoint.

**Risk:** If Google's CDN is compromised (low probability, high impact), the CSS response could inject malicious `@font-face` rules or even exploit CSS-based data exfiltration techniques. More practically, Google tracks every visitor via this request (IP address, user-agent, referrer).

**Remediation options (choose one):**

| Option | Privacy | Performance | Effort |
|---|---|---|---|
| **Self-host fonts** (recommended) | No third-party tracking | Faster (same origin) | Medium — download WOFF2 files, add `@font-face` rules |
| Keep Google Fonts + `crossorigin` attribute | Google still tracks | Same | Low — add `crossorigin="anonymous"` to the link tag |
| Use `font-display: swap` with system fallbacks | Full privacy | Fastest | Medium — design for font fallbacks |

**Strong recommendation: Self-host Orbitron and Saira.** Download WOFF2 files from Google Fonts, place in `dist/assets/fonts/`, and use `@font-face` with `font-display: swap`. This eliminates the only third-party runtime dependency.

---

### FINDING 6 — `.DS_Store` Files in Deployment Directory
**Severity: MEDIUM**
**Category: Information Disclosure**

```
dist/.DS_Store          (6,148 bytes)
tarvan_webiste/.DS_Store (10,244 bytes)
```

**Risk:** `.DS_Store` files are macOS Finder metadata files. They contain directory listing information — file names, folder structure, icon positions. If deployed to the web server, they can be accessed directly at `https://tarvan.ai/.DS_Store` and parsed to reveal the full directory structure. Tools like `ds_store` (Python) can extract this information trivially.

This is a known, actively-exploited information disclosure vector. Pentesters routinely check for it.

**Remediation:**

1. Delete from `dist/`: `rm dist/.DS_Store`
2. Add to `.gitignore`: `**/.DS_Store`
3. If using a hosting provider with deploy config, exclude `.DS_Store` in the build step
4. For Cloudflare Pages / Netlify, add to the ignored files list

---

### FINDING 7 — innerHTML Used for Logo Rendering
**Severity: LOW (in current context)**
**Category: XSS Vector**

Six `innerHTML` assignments in the JS:

```javascript
document.getElementById('nav-logo').innerHTML = mono(36, { stroke: LOGO_STROKE });
document.getElementById('hero-logo').innerHTML = mono(80, { stroke: LOGO_STROKE });
// ...etc
```

**Current risk: NONE.** The `mono()` function generates SVG from hardcoded constants — no user input enters the rendering pipeline. All arguments are string literals or computed from numeric constants.

**Future risk: LOW.** If someone later modifies `mono()` to accept user-controllable input (e.g., query params, URL hash), this becomes a direct DOM XSS injection point.

**Remediation (defense-in-depth):**

No immediate action required. But document this as a rule:

> **SECURITY INVARIANT:** The `mono()` function must never accept user-controllable input. All arguments must be hardcoded constants or derived from constants.

For a future refactor, consider using `document.createElement` + DOM API instead of `innerHTML` for SVG generation.

---

### FINDING 8 — Smooth Scroll Uses querySelector with href Value
**Severity: LOW**
**Category: DOM-based XSS (theoretical)**

```javascript
var href = this.getAttribute('href');
// ...
target = document.querySelector(href);
```

**Current risk: NONE.** The `querySelectorAll('a[href^="#"]')` pre-filter ensures only `#`-prefixed hrefs enter this path, and all anchors are hardcoded in the HTML. The `try/catch` around `querySelector` prevents CSS selector injection from crashing the page.

**Theoretical risk:** If a future developer adds an anchor with a `href` containing CSS selector syntax beyond a simple `#id`, `querySelector` could behave unexpectedly. The `try/catch` mitigates this.

**Remediation:** No action required. The existing `try/catch` is sufficient defense for a static site.

---

### FINDING 9 — Founder Email Exposed as Plaintext
**Severity: LOW-MEDIUM**
**Category: Spam / Abuse**

```html
<a href="mailto:shan@tarvan.ai">shan@tarvan.ai</a>
```

The founder's email appears twice in plaintext HTML — in the CTA section and the footer. Web scrapers harvest `mailto:` links aggressively.

**Risk:** Increased spam to the founder's inbox. For a B2B site this is a deliberate trade-off (direct email builds trust with OEM buyers), but the spam cost is real.

**Remediation options:**

| Option | Trade-off |
|---|---|
| Keep as-is (recommended for B2B) | Accept spam; use email filtering |
| Light obfuscation via JS | `shan` + `@` + `tarvan.ai` assembled in JS — blocks simple scrapers |
| Contact form (Formspree, Netlify Forms) | Lower spam; higher friction for prospects |
| CloudFlare Email Address Obfuscation | If hosted on CF Pages, enable this — it's automatic and free |

**Recommendation:** If deploying on Cloudflare Pages, enable their automatic email obfuscation. Otherwise, keep as-is — the B2B trust benefit outweighs the spam cost, and a spam filter handles the rest.

---

### FINDING 10 — No `security.txt` File
**Severity: LOW**
**Category: Security Hygiene / Responsible Disclosure**

No `/.well-known/security.txt` file exists. This is the standard mechanism for security researchers to report vulnerabilities.

**Risk:** If someone finds a vulnerability in tarvan.ai (e.g., a DNS misconfiguration, a CDN issue), they have no structured way to report it. They might disclose publicly instead.

**Remediation — create `dist/.well-known/security.txt`:**

```
Contact: mailto:shan@tarvan.ai
Preferred-Languages: en
Canonical: https://tarvan.ai/.well-known/security.txt
Expires: 2027-03-07T00:00:00.000Z
```

---

### FINDING 11 — No `robots.txt` or `sitemap.xml`
**Severity: LOW**
**Category: Information Hygiene**

No `robots.txt` to control crawler behavior. No `sitemap.xml` for search engine indexing.

**Risk:** Low direct security risk. But without `robots.txt`, all paths are crawlable by default — including any accidentally deployed files (the `.DS_Store` issue above). Also, missing `sitemap.xml` hurts SEO.

**Remediation — create `dist/robots.txt`:**

```
User-agent: *
Allow: /
Disallow: /assets/

Sitemap: https://tarvan.ai/sitemap.xml
```

**Create `dist/sitemap.xml`:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://tarvan.ai/</loc>
    <lastmod>2026-03-07</lastmod>
    <priority>1.0</priority>
  </url>
</urlset>
```

---

### FINDING 12 — Source Files in Deployment Directory
**Severity: LOW**
**Category: Information Disclosure**

The `files/` directory contains:

```
COWORK-PROMPT.md          — CoWork prompt instructions
INSTRUCTIONS.md           — Build instructions
TARVAN_Final_Production_Lockups.html  — Brand lockup source
brand-spec.md             — Full brand specification
dark-mockup.html          — Design mockup
```

**Risk:** If the entire `tarvan_webiste/` folder is deployed (not just `dist/`), these files become publicly accessible. They contain internal brand specifications, build instructions, and AI prompt engineering details — not something you want competitors reading.

**Remediation:**

1. Deploy ONLY the `dist/` folder — never the parent
2. Add a `.gitignore` or deploy config that explicitly excludes `files/`, `webpage-mockups/`, `*.md` (at root level)
3. Verify this after the first deployment by checking `https://tarvan.ai/files/brand-spec.md` returns 404

---

## Summary Table

| # | Finding | Severity | Status | Effort to Fix |
|---|---|---|---|---|
| 1 | No Content Security Policy | **HIGH** | Open | 5 min — add meta tag |
| 2 | No X-Frame-Options | MEDIUM | Open | 1 min — add meta tag |
| 3 | No Referrer-Policy | MEDIUM | Open | 1 min — add meta tag |
| 4 | No Permissions-Policy | LOW | Open | 1 min — add meta tag |
| 5 | Google Fonts without SRI / privacy leak | MEDIUM | Open | 30 min — self-host fonts |
| 6 | `.DS_Store` in deploy dir | MEDIUM | Open | 1 min — delete files |
| 7 | innerHTML for logo rendering | LOW | Accepted | N/A — document invariant |
| 8 | querySelector with href | LOW | Accepted | N/A — try/catch is sufficient |
| 9 | Founder email in plaintext | LOW-MEDIUM | Accepted | Use CF obfuscation if available |
| 10 | No security.txt | LOW | Open | 5 min — create file |
| 11 | No robots.txt / sitemap.xml | LOW | Open | 5 min — create files |
| 12 | Source files in parent dir | LOW | Open | Deploy config — dist/ only |

---

## Recommended Priority Order

**Immediate (before launch):**

1. Add CSP meta tag (Finding 1)
2. Add X-Frame-Options meta tag (Finding 2)
3. Add Referrer-Policy meta tag (Finding 3)
4. Add Permissions-Policy meta tag (Finding 4)
5. Delete `.DS_Store` files (Finding 6)
6. Verify deployment scope is `dist/` only (Finding 12)

**Before or shortly after launch:**

7. Self-host Google Fonts (Finding 5)
8. Create `robots.txt` and `sitemap.xml` (Finding 11)
9. Create `security.txt` (Finding 10)

**Accepted / deferred:**

10. innerHTML pattern (Finding 7) — document the invariant
11. querySelector pattern (Finding 8) — no action needed
12. Email obfuscation (Finding 9) — enable CF obfuscation or accept

---

## 8-Point Review Checklist

1. **Correctness** — All findings verified against source code. No false positives.
2. **Tests** — N/A for audit. Remediation can be verified by running `curl -I https://tarvan.ai/` after deployment and checking response headers, or by inspecting the HTML `<head>`.
3. **Security** — This IS the security review. 12 findings, 1 HIGH, 3 MEDIUM.
4. **Privacy** — Google Fonts privacy leak identified (Finding 5). Email exposure documented (Finding 9).
5. **Safety** — N/A for static website.
6. **Performance** — Self-hosting fonts (Finding 5 remediation) will improve performance as a side-effect.
7. **Readability** — Report structured for actionability. Findings ordered by severity.
8. **Scope** — Audit covers the deployed artifact (`dist/`) only. Infrastructure-level security (DNS, hosting platform, TLS configuration) is out of scope and should be audited separately after deployment.
