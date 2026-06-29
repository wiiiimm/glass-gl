# Glass UI — Plain CSS

A token-driven system. Define the variables once, then every glass surface is one class.

## Token base

```css
:root {
  --glass-tint: 255 255 255;          /* RGB channels (space-separated for rgb()/alpha) */
  --glass-fill: 0.10;                  /* fill alpha */
  --glass-blur: 22px;
  --glass-saturate: 1.2;
  --glass-border: rgba(255 255 255 / 0.30);
  --glass-highlight: rgba(255 255 255 / 0.35);  /* inset top lip */
  --glass-shadow: 0 24px 60px rgba(0 0 0 / 0.18);
  --glass-radius: 24px;
}

.glass {
  background: rgb(var(--glass-tint) / var(--glass-fill));
  backdrop-filter: blur(var(--glass-blur)) saturate(var(--glass-saturate));
  -webkit-backdrop-filter: blur(var(--glass-blur)) saturate(var(--glass-saturate));
  border: 1px solid var(--glass-border);
  border-radius: var(--glass-radius);
  box-shadow: var(--glass-shadow), inset 0 1px 0 var(--glass-highlight);
  color: #fff;
}
```

## Variants (override tokens per element)

```css
/* Card / panel — the default above is already this */

/* Button: smaller radius, lighter, brighten on hover */
.glass-btn {
  background: rgb(var(--glass-tint) / 0.07);
  backdrop-filter: blur(12px) saturate(var(--glass-saturate));
  -webkit-backdrop-filter: blur(12px) saturate(var(--glass-saturate));
  border: 1px solid rgba(255 255 255 / 0.45);
  border-radius: 14px;
  color: #fff;
  padding: 0 22px; height: 48px;
  cursor: pointer;
  transition: background .15s ease, transform .12s ease;
}
.glass-btn:hover  { background: rgb(var(--glass-tint) / 0.16); }
.glass-btn:active { transform: translateY(1px); }

/* Pill input */
.glass-input {
  background: rgb(var(--glass-tint) / 0.06);
  border: 1px solid rgba(255 255 255 / 0.45);
  border-radius: 999px;
  padding: 0 16px; height: 46px;
  color: #fff;
}
.glass-input::placeholder { color: rgba(255 255 255 / 0.75); }

/* Sticky navbar: lighter blur so it stays fast on scroll */
.glass-nav {
  position: sticky; top: 0;
  background: rgb(var(--glass-tint) / 0.08);
  backdrop-filter: blur(14px) saturate(1.3);
  -webkit-backdrop-filter: blur(14px) saturate(1.3);
  border-bottom: 1px solid rgba(255 255 255 / 0.18);
}

/* Dark glass — for placing over LIGHT backgrounds */
.glass-dark {
  --glass-tint: 10 12 20;
  --glass-fill: 0.35;
  --glass-border: rgba(255 255 255 / 0.12);
  --glass-highlight: rgba(255 255 255 / 0.10);
}
```

## Minimal HTML

```html
<div class="glass" style="padding:24px; max-width:360px">
  <h2>Frosted</h2>
  <p>Place me over a photo or gradient.</p>
  <button class="glass-btn">Action</button>
</div>
```

## Notes
- `rgb(R G B / A)` (space syntax) lets one `--glass-tint` triple drive both the fill alpha and any derived colors. Supported in all current browsers; if you must support very old ones, fall back to comma `rgba()` with hardcoded values.
- Keep blur radius lower on large, scroll-pinned surfaces (navbars) than on small static cards.
- See `../SKILL.md` for the mandatory `@supports` / `prefers-reduced-transparency` fallbacks.
