# TARVAN Website — Design Critique

**Critique type:** Code-informed structural design review (no live render — based on full CSS/HTML analysis)
**Date:** 2026-03-07

---

## 1. First Impression (The 2-Second Test)

### What draws the eye first?

On desktop ≥1360px, the hero presents a split composition: left-aligned text block (badge → lockup → description → CTAs) and a right-positioned orbital animation (520×520px, at `right: 4%`). The L1 lockup — 80px heptagon + "TARVAN" in `clamp(36px, 6vw, 56px)` Orbitron — is the visual anchor of the left half. The orbital with its orange animated dots and breathing rings pulls attention to the right. This is a good tension — the eye bounces between brand identity (left) and brand metaphor (right).

On the 900–1360px range, the hero content is constrained by the `.container` (1200px centered, 48px padding). The orbital shrinks but remains visible. The composition holds.

On mobile (≤900px), the orbital **vanishes** entirely (`display: none`). The first impression becomes: orange badge pill → large wordmark → gray description → two buttons. Clean, but the distinctive visual identity is gone. The hero becomes functionally identical to any modern startup landing page.

### Emotional reaction?

Dark, confident, technical. The navy-to-surface color rhythm with orange accents at ≤5% reads as "serious industrial technology" — which is exactly right for Physical AI. The breathing orbital animation adds a sense of intelligence-in-motion without being flashy. The grid overlay with its barely-visible orange lines and mask gradient creates depth without clutter.

### Is the purpose immediately clear?

Yes. "Physical AI Platform" badge → "TARVAN" → "Intelligence That Moves Matter." → description mentioning "machine intelligence layer for industrial OEMs" — the value proposition chain is tight and correctly sequenced. A visiting OEM executive would know within 2 seconds what this company does.

---

## 2. Usability

### Can the user accomplish their goal?

The primary conversion path is clear: "Start a Pilot" (primary orange CTA) and "Talk to the Founder" (bottom CTA, `mailto:shan@matic.one`). The nav CTA "Talk to Us" is also orange, creating a persistent call-to-action. For a B2B site where the goal is "generate a qualified conversation," this is well-structured.

**Concern:** Both CTAs link to `#cta` or `mailto:`. There's no form — every engagement requires an email. For some prospects, especially those in early exploration, a form with a lower commitment threshold might convert better. That said, for a deep-tech company selling to OEMs, founder-direct email is appropriate and may actually increase trust.

### Is the navigation intuitive?

Six items: Framework, Platform, Verticals, OEM, Credentials, Talk to Us. This maps directly to the buyer's information journey — "what is it → how does it work → where does it apply → how do I integrate → why should I trust you → let's talk." The ordering is intentional and correct.

The nav items are 13px uppercase Saira 500 with 0.06em tracking — slightly small but appropriate for a sparse, high-end navigation bar. The orange underline hover effect (1.5px, width transitions from 0→100%) is subtle and elegant.

**What works:** Sticky nav with frosted glass on scroll (`backdrop-filter: blur(20px)`, transition from transparent to 92% opacity navy). This is a well-executed pattern that maintains context without being heavy.

### Are interactive elements obvious?

Buttons use clear visual hierarchy: primary (filled orange, 16px padding, arrow icon with nudge animation) vs. secondary (ghost with white border). The hover states are distinct — primary lifts with box-shadow, secondary changes border/text to orange. Good.

**Issue:** The nav link hover underline animation (0→100% width, left-to-right) uses `width` for animation, which triggers layout repaints. Should use `transform: scaleX()` for compositor-only animation. This is a micro-performance concern, not a usability one, but worth noting.

---

## 3. Visual Hierarchy

### Reading order

The vertical flow is well-structured:

1. **Hero** — identity + value prop + CTA (100vh)
2. **Metrics** — proof bar (4 hard numbers, no fluff)
3. **Sense·Think·Act** — framework explanation (3-card grid)
4. **Platform** — technical depth (features + architecture diagram)
5. **Verticals** — market application (3-card grid)
6. **OEM** — integration story (machine mockup + benefits list)
7. **Credentials** — trust signals (institutions + grants)
8. **Founder** — personal credibility
9. **CTA** — conversion close
10. **Footer** — navigation + legal

This follows the classic B2B SaaS storytelling arc: hook → proof → explain → apply → partner → trust → convert. Each section has a consistent internal rhythm: badge → title → description → content grid. This repetition creates predictability without monotony.

### Are the right elements emphasized?

**Section titles:** `clamp(28px, 4vw, 40px)` Saira 700 — appropriately dominant. The section badges (10px uppercase, orange, letter-spacing 0.2em) serve as gentle category labels. This two-tier heading system is clean.

**Metrics bar:** Orbitron font for numbers creates visual distinction. **However** — as noted in the audit, the brand spec restricts Orbitron to the wordmark only. The metrics work visually, but this is a brand compliance issue. Saira 800 at the same size would still be striking without breaking the type hierarchy.

**CTA box:** The gradient border (`border: 2px solid var(--orange)`) + gradient background + radial gradient glow is the most visually "hot" element below the hero. This is correct — the final conversion point should feel like the culmination.

### Whitespace

Section padding is 120px vertically (top + bottom), which gives generous breathing room between content blocks. The `.container` at 1200px with 48px padding creates comfortable line lengths (roughly 65-70 characters per line at 18px body text). The internal grids use 40–48px gaps.

**One concern:** The hero description (`max-width: 580px`, 18px, line-height 1.7) sits well, but the 48px margin-bottom between description and CTA buttons feels slightly loose. 32–36px might create a tighter visual connection between the pitch and the action.

### Typography hierarchy

The type scale is well-structured:

| Element | Size | Weight | Font | Tracking |
|---------|------|--------|------|----------|
| Wordmark | clamp(36-56px) | 700 | Orbitron | 0.08em |
| Section title | clamp(28-40px) | 700 | Saira | — |
| Body/description | 17-18px | 400 | Saira | — |
| Labels/badges | 10-12px | 600-700 | Saira | 0.12-0.2em |
| Nav links | 13px | 500 | Saira | 0.06em |

The scale has clear steps with no collisions. Saira's wide weight range (300-800) provides enough variation within a single family. The Orbitron/Saira pairing is distinctive — geometric display vs. humanist text — and creates a "tech + human" tension appropriate for the brand.

---

## 4. Consistency

### Design system adherence

Color usage is rigidly consistent: all values trace back to `:root` custom properties that match the brand spec exactly. No magic numbers for colors anywhere in the CSS. The border style (`rgba(255,255,255,0.06)`) is reused across nav, sections, footer, and cards. The orange accent appears only in: badges, CTAs, stat numbers, hover states, and the orbital — never as a background fill for large areas. The ≤5% rule is maintained.

### Spacing consistency

Section padding: 120px (hero uses `min-height: 100vh` instead). Card padding: 48px. Grid gaps: 32–48px. Button padding: 16px 32px. These values are consistent across all sections. The 48px is the dominant spacing unit, with 32px and 16px as subdivisions. This creates a coherent rhythm.

### Element behavior consistency

All cards (`.sta-card`, `.vertical-card`, `.cred-card`) share:
- Same background (`var(--navy-mid)`)
- Same border (`1px solid var(--border)`)
- Same border-radius (`16px`)
- Same hover behavior (border color changes to `rgba(232,119,34,0.2)` with translateY(-2px))
- Same reveal animation staggering

This consistency is excellent. A user understands "this is a card" immediately upon encountering a new section.

**One inconsistency:** The OEM section uses `.oem-machine-mockup` with slightly different styling (background: `rgba(15,29,51,0.5)` instead of solid `var(--navy-mid)`, dashed border). This deliberate differentiation works — it signals "this is a demo/mockup, not a content card" — but it slightly breaks the card pattern.

---

## 5. Accessibility (Design-Level)

### Color contrast

| Text | Color | Background | Ratio | AA Pass? |
|------|-------|------------|-------|----------|
| Body text | #E8ECF1 | #0A1628 | ~13.5:1 | Yes |
| Description text | #8A96A8 | #0A1628 | ~4.8:1 | Yes (barely) |
| Dim text | #5A6577 | #0A1628 | ~3.1:1 | **No** |
| Orange on navy | #E87722 | #0A1628 | ~4.6:1 | Yes (barely for normal, yes for large) |
| Button text | #0A1628 | #E87722 | ~4.6:1 | Yes (barely) |

The gray-dim (`#5A6577`) elements — scroll indicator label, footer legal text, footer bottom copy — fail AA contrast. These are secondary/tertiary text, but if they're meant to be read (and footer legal text should be), they need to be lighter.

### Touch targets

Button padding (16px 32px) creates comfortably sized touch targets (~44px height minimum). The nav links in mobile mode have 24px gap between them with full-width tap areas. The nav toggle is 32×24px — this is slightly below the recommended 44×44px minimum for mobile touch targets.

**What should change:** Increase `.nav-toggle` to at least 44×44px (add padding while keeping the visual hamburger at current size).

### Text readability

Body text at 18px with line-height 1.7 on a max-width container creating ~65-character lines is excellent for readability. The Saira typeface has good x-height and open counters, making it legible even at smaller sizes.

**Concern:** The orbital labels ("Sense", "Think", "Act") at 12px with 0.2em tracking are decorative, but if a user tries to read them they're quite small relative to the 520px orbital. At this size, they serve more as compositional elements than readable text — which is acceptable for the hero visual.

---

## 6. What Works Well (Positive Observations)

**Motion as brand expression.** The hero motion system — idle drift, pointer parallax, orbital breathing, staggered entrance — doesn't just look good. It embodies the brand promise: intelligence that *moves*. The three orbital rings with Sense/Think/Act labels create an immediate visual connection to the core framework. This is design serving strategy, not decoration.

**Metrics bar as proof-first design.** Placing hard numbers (<85ms, 10/sec, 4-lane, 0%) immediately below the hero is a strong B2B move. It says "we have numbers, not just claims." The compact, high-density format respects the reader's time.

**Architecture diagram in the Platform section.** The stacked layer diagram (Application → Inference → Edge Runtime → Hardware IO) is simple and effective. It communicates the full-stack nature of TARVAN without requiring a complex Mermaid diagram. The graduated colors from navy-light (top) to orange-accented (bottom) create a natural reading flow.

**OEM machine mockup.** The minimal SVG mockup of a machine front panel with the "Powered by TARVAN" badge is clever. It contextualizes the ingredient-brand positioning without requiring a photo. For a company that may not yet have product photography, this is a smart design solution.

**Credential cards.** The simple two-column layout (Incubation | Grants) with institutional names avoids the common trap of logo-soup credential sections. Text-only is actually more credible here — it signals "we don't need to borrow authority from logos."

---

## 7. Key Recommendations (Design-Level)

### Recommendation 1: Restore hero visual on mobile
The orbital animation is the most distinctive design element. Hiding it entirely on mobile removes the "Physical AI" visual identity from the majority of visitors. A 280px scaled-down version centered behind the hero text (40% opacity, no pointer parallax, static rings acceptable) would preserve brand expression without performance cost.

### Recommendation 2: Tighten hero description-to-CTA spacing
The 48px gap between `.hero-desc` and `.hero-actions` feels slightly disconnected. Reducing to 32px would create a stronger visual link between the pitch paragraph and the action buttons, improving conversion proximity.

### Recommendation 3: Add a lightweight social proof element
The metrics bar provides numerical proof, but there's no customer/user signal (even anonymous). A single line like "Deployed in 3 processing facilities" or "Grading 40 kernels/sec in production" between the metrics bar and the framework section would add credibility without requiring a full testimonial section.

### Recommendation 4: Consider a dark/light mode toggle or auto-detection
The entire site is dark-only. For users browsing in bright environments (factory floor with daylight, office with overhead lighting), a dark theme can be harder to read. A subtle toggle in the footer or nav could serve accessibility-conscious visitors. Low priority but thoughtful.

### Recommendation 5: Fix nav toggle touch target
Increase from 32×24px to 44×44px minimum. This is a quick CSS fix that meaningfully improves mobile usability.

---

*This critique complements the technical audit report (tarvan-website-audit-v1.md). Findings that overlap — OG image, contrast ratios, Orbitron scoping — are referenced but not duplicated here.*
