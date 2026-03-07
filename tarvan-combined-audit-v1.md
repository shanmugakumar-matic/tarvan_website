# TARVAN Website — Combined Pre-Launch Audit

**Date:** 2026-03-08
**Auditors:** Claude (Opus 4.6) + Developer Review
**File:** `dist/index.html` (114 KB, single-page static)

---

## How to Read This Report

Every finding is tagged with its source:

- **[BOTH]** — Caught by both audits. High confidence. Fix immediately.
- **[DEV]** — Caught only by developer. Claude missed it.
- **[CLAUDE]** — Caught only by Claude. Developer missed it.

Severity: **P0** = launch blocker, **P1** = fix before launch, **P2** = fix within first week.

---

## Findings Summary

| # | Finding | Severity | Source | Est. Time |
|---|---|---|---|---|
| 1 | Unverifiable institutional credentials | P0 | DEV | 15 min |
| 2 | Compliance claims overstate readiness | P0 | DEV | 20 min |
| 3 | Gray text fails WCAG contrast | P0 | CLAUDE | 10 min |
| 4 | Orange text fails WCAG at small sizes | P0 | CLAUDE | 20 min |
| 5 | SVG min-width:700px breaks mobile | P0 | BOTH | 15 min |
| 6 | Missing keyboard focus indicators | P0 | CLAUDE | 20 min |
| 7 | Hard performance numbers lack context | P1 | BOTH | 20 min |
| 8 | Orbit dots ignore reduced-motion | P1 | DEV | 15 min |
| 9 | OEM carousel lacks accessible pause/keyboard | P1 | DEV | 30 min |
| 10 | Meta security headers need HTTP layer | P1 | DEV | Deploy task |
| 11 | Missing JSON-LD structured data | P1 | CLAUDE | 30 min |
| 12 | Unoptimized images (187KB + 821KB) | P1 | CLAUDE | 15 min |
| 13 | Inconsistent "Physical AI" terminology | P1 | CLAUDE | 30 min |
| 14 | "India's deepest" superlative unsubstantiated | P1 | DEV | 5 min |
| 15 | Heading hierarchy — no H3s | P2 | CLAUDE | 20 min |
| 16 | Brand color deviations (orange + navy) | P2 | CLAUDE | Decision |
| 17 | Missing mobile breakpoints below 480px | P2 | CLAUDE | 30 min |
| 18 | Hero orbital not hidden from assistive tech | P2 | DEV | 5 min |
| 19 | robots.txt contradictions | P2 | CLAUDE | 5 min |
| 20 | Inline onerror on founder photo | P2 | BOTH | 5 min |

**Total estimated remediation: ~5 hours** (P0 + P1 in Day 1, P2 in Day 2)

---

## P0 — LAUNCH BLOCKERS

### 1. [DEV] Unverifiable Institutional Credentials
**Lines 1805–1816**

The Credentials section claims: SIIC IIT Kanpur, NSRCEL IIM Bangalore, ICAR-CISH, CCS NIAM, DISQ (TCS Foundation). The developer could only publicly verify the MaDeIT / IIITDM connection. Unverifiable institutional backing reads as overclaim even if internally true.

**Action required (Dr. Shan — decision needed):**
- Option A: Add proof links (incubation certificates, program pages) next to each claim
- Option B: Keep only publicly verifiable ones (MaDeIT/IIITDM), remove the rest
- Option C: Reframe as "Applied to" or "Engaged with" rather than implying formal backing

---

### 2. [DEV] Compliance Claims Overstate Formal Readiness
**Lines 1220, 1251, 1262, 1280, 1792–1793**

Multiple locations overstate compliance status:

| Line | Current Claim | Problem |
|---|---|---|
| 1220 | "Compliance-ready audit logs for FSSAI, ISO, export, and industrial safety certification" | Implies certification readiness without evidence |
| 1262 | "FSSAI" + "Ready" | Implies FSSAI certification readiness |
| 1280 | "ISO" + "Compliance" | Implies ISO compliance achieved |
| 1793 | "Your customers pass audits with data, not paperwork" | Guarantees audit outcomes |

**Fix:** Reframe as capability, not certification. Examples:
- "Structured audit logs that support FSSAI, ISO, and export documentation workflows"
- "Traceability-ready" instead of "FSSAI Ready"
- "Supports ISO documentation" instead of "ISO Compliance"
- "Helps your customers approach audits with data" instead of "pass audits"

---

### 3. [CLAUDE] Gray Text Fails WCAG AA Contrast
**CSS variable `--gray: #8A96A8` on `--navy: #0A1628`**

Contrast ratio: **2.87:1** (requires 4.5:1 for AA).
Affects virtually all body text: nav links, hero description, section descriptions, footer, bio.

**Fix:** Change `--gray` to `#B0BEC5` or `#B8C4D4` (achieves ~5:1 on navy).

---

### 4. [CLAUDE] Orange Text Fails WCAG at Small Sizes
**`#E87722` on `#0A1628`**

Contrast ratio: **3.42:1**. Passes for large text (18px+ bold) but fails for normal text. Multiple labels in SVG diagrams at 10–12px use this orange directly.

**Fix:** Either increase all orange text to 18px+ bold, or use a lighter orange (`#FF9245`, ~5.1:1) for text below 18px.

---

### 5. [BOTH] SVG `min-width:700px` Breaks Mobile
**Lines 1292, 1441, 1611 — all 3 OEM carousel SVGs**

`style="min-width:700px"` forces horizontal scroll on any device narrower than 700px — every phone.

**Fix:** Remove the inline `min-width`. Add CSS:
```css
@media (max-width: 768px) {
  .oem-slide { overflow-x: auto; -webkit-overflow-scrolling: touch; }
  .oem-slide svg { min-width: 600px; }
}
```

---

### 6. [CLAUDE] Missing Keyboard Focus Indicators
**No `:focus-visible` styles anywhere in CSS**

Keyboard-only users cannot see which element is focused. CSS has `outline: none` on main but no replacement focus ring.

**Fix:** Add:
```css
a:focus-visible, button:focus-visible, .btn-primary:focus-visible, .btn-secondary:focus-visible {
  outline: 2px solid #E87722; outline-offset: 3px;
}
```

---

## P1 — FIX BEFORE LAUNCH WEEK

### 7. [BOTH] Hard Performance Numbers Lack Context
**Lines 1093–1108**

`<85ms`, `100% Edge Compute`, `0% Cloud Dependency` — presented as universal facts without specifying machine class, commodity, deployment, or test scope. "4 lanes" is cashew-specific.

**Developer's take:** "These are the first numbers a technical buyer will challenge."
**Claude's take:** "100% Edge" may not cover analytics/OTA/monitoring.

**Fix options:**
- Add fine print: "Measured on MATIC Cashew at 10 kernels/sec, Jetson Orin Nano"
- Or soften: "100% Edge Inference" + "Zero Cloud Dependency for Operation"
- Change "4 Industry Verticals" (this is fine as-is — it's a count, not a claim)

---

### 8. [DEV] Orbit Dots Ignore `prefers-reduced-motion`
**Lines 1010–1048**

The hero section respects reduced-motion for CSS animations and RAF-driven motion, but the 7 orbit dots use SVG `<animateMotion>` (SMIL animation) which is NOT stopped by the CSS media query. Motion-sensitive users still see continuous animation.

**Fix:** Add:
```css
@media (prefers-reduced-motion: reduce) {
  .orbit-dot animateMotion, animateMotion { display: none; }
}
```
Or alternatively, use JS to remove `<animateMotion>` elements when `matchMedia('(prefers-reduced-motion: reduce)')` matches.

---

### 9. [DEV] OEM Carousel Lacks Accessible Pause/State
**Lines ~2214–2258 (JS carousel logic)**

Three accessibility gaps:
- No visible pause/play button (WCAG 2.2.2 requires user control of auto-playing content)
- Dot buttons have no `aria-selected` or active-state announcement
- Auto-rotation doesn't stop for reduced-motion or keyboard-only users

**Fix:**
- Add a pause/play toggle button in `.oem-carousel-dots`
- Set `aria-selected="true"` on active dot
- Stop auto-rotation when `prefers-reduced-motion: reduce`
- Stop auto-rotation on keyboard focus within carousel

---

### 10. [DEV] Security Headers Must Move to HTTP Layer
**Lines 24–28**

CSP, X-Frame-Options, Referrer-Policy, and Permissions-Policy are currently delivered as `<meta>` tags. This is better than nothing, but:
- HSTS **cannot** be delivered from HTML at all
- X-Frame-Options as meta tag is not reliable across all browsers
- `unsafe-inline` remains necessary for the inline CSS/JS approach

**Fix:** This is a **deployment task for Cloudflare**, not an HTML task:
- Set response headers in Cloudflare dashboard or `_headers` file
- Add `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- Keep meta tags as fallback for file:// preview

---

### 11. [CLAUDE] Missing JSON-LD Structured Data

No `<script type="application/ld+json">` anywhere. Google cannot build rich snippets or knowledge graph entries.

**Fix:** Add Organization schema with: name, url, logo, founder, sameAs (LinkedIn), description, contactPoint.

---

### 12. [CLAUDE] Unoptimized Images

| File | Current | Target |
|---|---|---|
| `Dr.Shanmugakumar_Photo.jpeg` | 187 KB | ~30–50 KB (320x320, quality 80) |
| `og-card.png` | 821 KB | ~200–300 KB (compressed) |

**Fix:** Resize founder photo to 320x320 (2x retina for 160px display). Compress OG card with TinyPNG or `pngquant`.

---

### 13. [CLAUDE] Inconsistent "Physical AI" Terminology

Hero badge: "Physical AI Platform." Hero description: "machine intelligence layer." Section titles alternate between them. Some places say "intelligence stack."

**Fix:** Standardize primary term across all instances. Suggested: "Physical AI" as the category, "intelligence layer" as what gets integrated. Never "machine intelligence layer" standalone.

---

### 14. [DEV] "India's deepest" Superlative Unsubstantiated
**Line 1835**

"Building India's deepest Physical AI stack" — superlative claim that's unverifiable and invites challenge.

**Fix:** Replace with: "Building a vertically integrated Physical AI stack for industrial machines — from agri-processing to process automation."

---

## P2 — FIX WITHIN FIRST WEEK

### 15. [CLAUDE] Heading Hierarchy Missing H3s
Seven `<h2>` elements, zero `<h3>`. Screen readers and SEO crawlers expect proper nesting.

### 16. [CLAUDE] Brand Color Deviations
Site uses `#E87722` (orange) and `#0A1628` (navy). Brand spec says `#F5A623` and `#002366`. Either update to canonical or formally document the dark-theme variants.

### 17. [CLAUDE] Missing Mobile Breakpoints Below 480px
No CSS rules for screens below 480px. iPhone SE (375px) and Galaxy (360px) need dedicated handling.

### 18. [DEV] Hero Orbital Not Hidden from Assistive Tech
The decorative SVG orbital should have `aria-hidden="true"` and `role="presentation"` to hide from screen readers.

### 19. [CLAUDE] robots.txt Contradictions
`Allow: /` then `Disallow: /assets/` with individual Allow exceptions. Simplify to just allow everything except private paths.

### 20. [BOTH] Inline `onerror` on Founder Photo
Move the `onerror` JavaScript handler from inline HTML attribute to the JS block at the bottom of the page.

---

## What Passed Both Audits

Both auditors agreed the following are solid:

- Security posture (CSP, X-Frame-Options, Referrer-Policy, Permissions-Policy present)
- Self-hosted fonts (WOFF2, font-display: swap, zero external requests)
- Logo integrity (all inline SVG heptagons correct)
- Slogan consistency ("Intelligence That Moves Matter." with period, everywhere)
- No broken links, no placeholder text, no hardcoded secrets
- Clean HTML structure — no duplicate IDs, no broken markup
- Hero is more ownable than earlier versions
- Structure is clearer than previous iterations
- Reveal animations, H1, font, metadata issues from earlier sessions are fixed

---

## Gap Analysis — What Each Audit Missed

### Developer Missed (Claude Caught):
| Finding | Why It Matters |
|---|---|
| WCAG contrast failures (gray + orange) | Legal compliance, readability |
| Missing `:focus-visible` indicators | Keyboard accessibility |
| Missing JSON-LD schema | SEO discoverability |
| Image optimization (187KB + 821KB) | Mobile load time |
| Inconsistent terminology | Brand clarity |
| Missing <480px breakpoints | Small phone layout |
| robots.txt contradictions | Crawl efficiency |
| Heading hierarchy (no H3s) | SEO + screen readers |
| Brand color hex deviations | Brand integrity |

### Claude Missed (Developer Caught):
| Finding | Why It Matters |
|---|---|
| Unverifiable institutional claims | Credibility risk with OEMs |
| Compliance overclaims (FSSAI/ISO) | Legal liability, buyer trust |
| SVG animateMotion ignores reduced-motion | Accessibility for motion-sensitive users |
| Carousel lacks pause/aria-selected | WCAG 2.2.2 compliance |
| Meta headers vs HTTP response headers | Real security enforcement |
| Hero orbital needs aria-hidden | Screen reader noise |
| "India's deepest" superlative | Unsubstantiated claim |

---

## Recommended Fix Order

**Session 1 — Launch Blockers (P0, ~2 hours):**
1. Trim or substantiate credentials (Finding 1) — needs Dr. Shan decision
2. Reframe compliance claims (Finding 2) — copy changes
3. Fix `--gray` CSS variable for contrast (Finding 3)
4. Fix orange text sizes or color (Finding 4)
5. Fix SVG min-width for mobile (Finding 5)
6. Add :focus-visible styles (Finding 6)

**Session 2 — Pre-Launch Polish (P1, ~2.5 hours):**
7. Scope performance numbers (Finding 7)
8. Fix orbit dots reduced-motion (Finding 8)
9. Add carousel pause + aria states (Finding 9)
10. Compress images (Finding 12)
11. Add JSON-LD schema (Finding 11)
12. Standardize terminology (Finding 13)
13. Remove "India's deepest" (Finding 14)

**Session 3 — First Week (P2, ~1 hour):**
14–20. Remaining P2 items

**Deployment task (separate from code):**
- Cloudflare response headers (Finding 10)

---

*Combined from two independent audit passes. Total unique findings: 20.*
