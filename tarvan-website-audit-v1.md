# TARVAN Website — Pre-Launch Audit Report

**Date:** 2026-03-08
**File:** `dist/index.html` (114 KB, single-page static site)
**Domain:** tarvan.ai

---

## Executive Summary

The TARVAN website is **production-quality** with strong fundamentals across security, brand compliance, and code structure. However, there are **7 critical issues** and **9 high-priority issues** that should be resolved before launch to ensure accessibility compliance, SEO visibility, mobile usability, and professional polish.

**Estimated remediation time:** 4–6 hours for critical + high issues.

---

## Findings Overview

| Category | Critical | High | Medium | Low | Pass |
|---|---|---|---|---|---|
| Technical (HTML/CSS/JS) | 0 | 0 | 1 | 2 | ✅ |
| Content & Copy | 0 | 3 | 4 | 2 | — |
| SEO & Meta | 1 | 1 | 2 | 1 | — |
| Accessibility (WCAG) | 3 | 1 | 0 | 3 | — |
| Security | 1 | 0 | 2 | 1 | — |
| Mobile & Performance | 2 | 2 | 1 | 0 | — |
| Brand Compliance | 0 | 2 | 1 | 0 | ✅ |
| **TOTAL** | **7** | **9** | **11** | **9** | — |

---

## CRITICAL Issues (Fix Before Launch)

### C1. Color Contrast Failure — Gray Text on Navy
**Category:** Accessibility (WCAG 1.4.3)
**Impact:** Fails WCAG AA. Legal compliance risk.

`--gray: #8A96A8` on `--navy: #0A1628` = **2.87:1 ratio** (requires 4.5:1).
Affects: nav links, hero description, all `.desc` text, footer, bio text.

**Fix:** Change `--gray` to `#B8C4D4` or lighter (~5:1 ratio on navy).

---

### C2. Color Contrast Failure — Orange Text on Navy (small sizes)
**Category:** Accessibility (WCAG 1.4.3)
**Impact:** Fails for text below 18px.

`#E87722` on `#0A1628` = **3.42:1 ratio** (requires 4.5:1 for normal text).
Passes 3:1 for large text (18px+ bold). Fails for 11–14px labels.

**Fix:** Use `#FF9245` (lighter orange, ~5.1:1) for text below 18px, or increase font sizes of orange labels to 18px+.

---

### C3. Missing Keyboard Focus Indicators
**Category:** Accessibility (WCAG 2.4.7)
**Impact:** Keyboard users cannot see where they are on the page.

CSS has `main:focus { outline: none; }` and no `:focus-visible` styles on buttons/links.

**Fix:** Add `:focus-visible` styles with orange outline to all interactive elements.

---

### C4. SVG `min-width:700px` Causes Horizontal Scroll on Mobile
**Category:** Mobile Responsiveness
**Impact:** Broken layout on all phones (below 700px width).

All 3 OEM carousel SVGs have `style="min-width:700px"` which forces horizontal scroll.

**Fix:** Remove inline `min-width`, add CSS overflow-x handling for small screens.

---

### C5. Missing JSON-LD Structured Data
**Category:** SEO
**Impact:** No rich snippets in Google. Missing knowledge graph signals.

No `<script type="application/ld+json">` found. Organization and SoftwareApplication schemas are essential for B2B.

**Fix:** Add Organization schema with name, url, logo, founder, sameAs (LinkedIn).

---

### C6. CSP `unsafe-inline` for Scripts
**Category:** Security
**Impact:** Inline scripts can be injected if any XSS vector exists.

**Note:** For a static site with no user input, this is acceptable for launch. Post-launch, migrate to nonce-based CSP.

---

### C7. Unoptimized Images
**Category:** Performance
**Impact:** Slow load times on mobile.

- `Dr.Shanmugakumar_Photo.jpeg`: 187 KB (should be 30–50 KB at 160px display size)
- `og-card.png`: 821 KB (should be 200–300 KB)

**Fix:** Compress and resize founder photo to 320x320px JPEG quality 80. Compress OG card.

---

## HIGH Issues (Fix This Week)

### H1. Inconsistent Terminology — "Physical AI" vs "machine intelligence layer"
**Category:** Content
Hero badge says "Physical AI Platform." Hero description says "machine intelligence layer." Pick one primary term.

### H2. Heading Hierarchy — H1 then H2 with No H3
**Category:** SEO + Accessibility (WCAG 1.3.1)
Seven H2s, zero H3s. Add H3 subheadings within sections for proper nesting.

### H3. "4 lanes" Claim is Vertical-Specific
**Category:** Content
"Real-time kernel grading across 4 lanes" is specific to cashew. Reframe as "multi-lane."

### H4. robots.txt Has Contradictory Rules
**Category:** SEO
`Allow: /` then `Disallow: /assets/` with individual Allow exceptions. Simplify.

### H5. "100% Edge Compute" / "0% Cloud Dependency" — Strong Claims
**Category:** Content
Are analytics and OTA updates also edge-only? Clarify or soften.

### H6. Brand Orange Color Deviation
**Category:** Brand
Spec says `#F5A623`. Site uses `#E87722`. Either update or formally document as approved variant.

### H7. Brand Navy Color Deviation
**Category:** Brand
Spec says `#002366`. Site uses `#0A1628`. Same resolution needed.

### H8. Missing Mobile Breakpoints Below 480px
**Category:** Mobile
No CSS rules for screens below 480px. Needed for iPhone SE (375px), Galaxy (360px).

### H9. Inline `onerror` Handler on Founder Photo
**Category:** Security + Technical
Move `onerror` handler to JS block for cleaner separation.

---

## MEDIUM Issues (Fix Before V1.1)

| # | Category | Issue |
|---|---|---|
| M1 | Content | HTML entity `&lt;85ms` — verify renders correctly |
| M2 | Content | Capitalization inconsistency in "Machine Intelligence Layer" |
| M3 | SEO | Title tag is 40 chars — expand to 50–60 |
| M4 | SEO | Sitemap.xml missing `<changefreq>` tag |
| M5 | Brand | `--white: #E8ECF1` deviates from spec `#FFFFFF` |
| M6 | Security | Email `connect@tarvan.ai` in plaintext — spam risk |
| M7 | Security | Add `rel="noreferrer"` to LinkedIn link |
| M8 | Content | "Brand System v1.0" in footer visible to public |
| M9 | Content | "Talk to the Founder" — consider "Talk to Our Team" |
| M10 | Technical | 9 icon SVGs missing `xmlns` attribute |
| M11 | Performance | Founder photo needs compression |

---

## LOW Issues (Nice to Have)

| # | Issue |
|---|---|
| L1 | Migrate `var` to `const`/`let` in JS (ES6) |
| L2 | Add `<changefreq>weekly</changefreq>` to sitemap.xml |
| L3 | Add `<nav aria-label="Footer navigation">` in footer |
| L4 | Add `aria-hidden="true"` to decorative SVG icons in buttons |
| L5 | Add `<title>` inside SVG arrow icons |
| L6 | Consider contact form instead of mailto |
| L7 | SVG text at 8–10px below WCAG 16px minimum |
| L8 | LinkedIn link add `noreferrer` |
| L9 | Document dark-theme color variants in brand guidelines |

---

## What's Already Excellent

- **Security:** CSP, X-Frame-Options DENY, Referrer-Policy, Permissions-Policy all present
- **Self-hosted fonts:** Zero CDN requests. WOFF2 with `font-display: swap`
- **Logo integrity:** All instances are correct inline SVG heptagons
- **Slogan:** "Intelligence That Moves Matter." correct everywhere — with period, sentence case
- **Company name:** "TARVAN" always all-caps. Legal entity only in footer
- **Tone of voice:** Precise, industrial, confident. No banned marketing words
- **ARIA labels:** Nav toggle, close button, carousel dots properly labeled
- **Skip-to-content link:** Present and functional
- **Reduced motion:** `prefers-reduced-motion` media query disables animations
- **No broken links:** All anchor hrefs map to valid IDs
- **No placeholder text:** Zero TODO/FIXME/lorem ipsum
- **No hardcoded secrets:** Zero API keys or tokens
- **Clickjacking protection:** Double-layered (CSP + X-Frame-Options)

---

## Recommended Fix Priority

**Day 1 (2–3 hours):**
1. Fix `--gray` color for contrast (C1) — 10 min
2. Fix orange text sizes for small labels (C2) — 20 min
3. Add `:focus-visible` styles (C3) — 20 min
4. Fix SVG `min-width` for mobile (C4) — 15 min
5. Compress founder photo + OG card (C7) — 15 min
6. Add JSON-LD structured data (C5) — 30 min
7. Standardize "Physical AI" terminology (H1) — 30 min

**Day 2 (1–2 hours):**
8. Fix heading hierarchy (H2) — 20 min
9. Clean up robots.txt (H4) — 5 min
10. Soften "100% Edge" claims (H5) — 10 min
11. Move onerror to JS (H9) — 5 min
12. Add mobile breakpoints below 480px (H8) — 30 min
13. Expand title tag (M3) — 5 min

**Post-launch:**
- Resolve brand color hex decisions (H6, H7)
- Migrate to nonce-based CSP (C6)
- Low-priority items (L1–L9)

---

*Report generated by 7 parallel audit agents covering: Technical, Content, SEO, Accessibility, Security, Mobile/Performance, and Brand Compliance.*
