---
name: glass-ui
description: >-
  Build glassmorphism and liquid-glass UI — frosted translucent panels, cards,
  navbars, buttons, and modals — in any stack: plain CSS, SCSS, Tailwind, or
  shadcn/React, plus an advanced WebGL refraction-lens technique. Use whenever
  the user asks for frosted glass, glassmorphism, backdrop-blur surfaces,
  Apple-style "liquid glass", translucent cards/navbars/sidebars/modals, or a
  glass look on any element.
---

# Glass UI

**Three** techniques produce "glass", in rising order of fidelity and cost. Pick by how
much *refraction* you need, then by stack.

| Technique | Looks like | Cost | Use when |
|-----------|-----------|------|----------|
| **CSS glassmorphism** (`backdrop-filter: blur`) | Frosted, blurred panel that tints what's behind it | Cheap, 1 declaration block | 95% of cases: navbars, cards, modals, sidebars |
| **SVG displacement** (`feDisplacementMap`) | True refraction — background *bends* at the edges — with no canvas/WebGL | Cheap-ish; a `<filter>` (+ a hidden canvas only for the precise variant) | "Liquid glass" cards/tiles where you want bending, not just blur, but no WebGL |
| **WebGL liquid glass** (fragment shader) | The richest refraction + per-frame lens that tracks a dragging element | Expensive, needs a canvas + shader running every frame | Showpieces, draggable lenses, canvas→PNG export |

**Default to CSS glassmorphism.** Step up to **SVG displacement** when the user wants the
background to *bend/refract* (Apple-style "liquid glass") without a canvas. Reserve **WebGL**
for per-frame, interactive, or exportable showpieces.

> Note: most glass libraries in the wild are blur-only (tier 1). Tiers 2 and 3 are what make
> a glass implementation feel like physical glass — and are this skill's differentiator.

---

## The CSS recipe (memorize this)

Glass is **four layers stacked on a translucent fill**. Miss any one and it looks flat:

```css
.glass {
  /* 1. translucent fill — the "tint" of the glass */
  background: rgba(255, 255, 255, 0.10);
  /* 2. frost the content behind it (THE effect; everything else is polish) */
  backdrop-filter: blur(22px) saturate(1.2);
  -webkit-backdrop-filter: blur(22px) saturate(1.2);   /* Safari needs the prefix */
  /* 3. a bright hairline edge catches "light" on the rim */
  border: 1px solid rgba(255, 255, 255, 0.30);
  /* 4. drop shadow lifts it off the page + inset highlight = the glass "lip" */
  box-shadow:
    0 24px 60px rgba(0, 0, 0, 0.18),
    inset 0 1px 0 rgba(255, 255, 255, 0.35);
  border-radius: 24px;
}
```

### Three rules that decide whether it works

1. **There must be something busy/colorful BEHIND it.** Glass over a flat solid background is invisible. Place glass over a photo, gradient, or other content.
2. **`backdrop-filter` blurs what's *behind* the element**, not the element itself. (To blur the element's own content, use `filter`.) The element must be layered above other content (normal flow or `position` + `z-index`) for there to be a backdrop.
3. **Always ship the `-webkit-` prefix** and a fallback — see Accessibility & fallbacks below.

### Tuning knobs
- **Blur radius** (`blur(Npx)`): 8–14px = light frost; 20–30px = heavy frost. This is the single most important value.
- **Fill alpha** (`0.06`–`0.18`): higher = more opaque/milky, more legible text, less "glassy".
- **`saturate(1.1–1.4)`**: makes colors behind the glass pop — the trick that separates good glass from gray mush.
- **Dark glass** (for light backgrounds): tint with black instead — `background: rgba(0,0,0,0.25)` and use a lighter inset highlight sparingly.

---

## Choosing a stack

Read the matching reference file for ready-to-paste code, tokens, and variants (button / card / input / navbar):

- **Plain CSS** → `references/css.md` — custom-property token system + component variants
- **SCSS** → `references/scss.md` — a `glass()` mixin + `$glass-*` variables
- **Tailwind** → `references/tailwind.md` — utilities, arbitrary values, and a reusable `@utility`/plugin
- **shadcn / React** → `references/shadcn.md` — a `GlassCard` + `glass` variant via `cva` and `cn()`, using shadcn theme tokens
- **SVG displacement refraction** → `references/svg-displacement.md` — bend the background with `feDisplacementMap`, no WebGL (tier 2)
- **WebGL liquid glass** → `references/liquid-glass-webgl.md` — the refraction shader, distilled and annotated (tier 3)

Copyable stylesheet (framework-agnostic, all variants + fallbacks): `assets/glass.css`.

---

## Accessibility & fallbacks (don't skip)

```css
/* Fallback for browsers without backdrop-filter: lean on a more opaque fill */
@supports not ((backdrop-filter: blur(1px)) or (-webkit-backdrop-filter: blur(1px))) {
  .glass { background: rgba(30, 30, 40, 0.85); }
}
/* Respect users who reduce transparency (e.g. macOS/iOS setting) */
@media (prefers-reduced-transparency: reduce) {
  .glass { background: rgba(30, 30, 40, 0.92); backdrop-filter: none; -webkit-backdrop-filter: none; }
}
```

- **Contrast:** translucent glass kills text contrast. Bump fill alpha or add a subtle text shadow; verify text hits WCAG AA against the *busiest* part of the backdrop.
- **Performance:** `backdrop-filter` repaints on scroll and is GPU-heavy. Keep the number of simultaneous glass surfaces small; avoid animating blur radius.
- **WebGL:** gate the shader behind `prefers-reduced-motion` and provide the CSS-glass version as the fallback.

## Verifying
Glass is visual — confirm it by viewing, not by reading code. Serve the page (`python3 -m http.server`) over a colorful background and check: is the backdrop visibly blurred? is the rim highlight visible? does text stay legible? Test in Safari (prefix) and a reduced-transparency setting.
