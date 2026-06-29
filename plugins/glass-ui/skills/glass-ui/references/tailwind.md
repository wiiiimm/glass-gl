# Glass UI — Tailwind

## Option A — pure utilities (no config, fastest)

Tailwind ships `backdrop-blur-*`, `bg-white/N`, `border-white/N`, `shadow-*`. Compose them:

```html
<div class="rounded-3xl border border-white/30 bg-white/10
            backdrop-blur-xl backdrop-saturate-150
            shadow-[0_24px_60px_rgba(0,0,0,0.18)]
            [box-shadow:inset_0_1px_0_rgba(255,255,255,0.35)]
            p-6 text-white">
  Frosted card
</div>
```

- `backdrop-blur-xl` ≈ 24px. Use `backdrop-blur-md` (12px) for navbars.
- `backdrop-saturate-150` = the `saturate(1.5)` pop.
- The inset highlight has no first-class utility, so use the arbitrary `[box-shadow:...]` (or fold it into a component class, Option C).

## Option B — `@utility` (Tailwind v4)

```css
/* app.css */
@import "tailwindcss";

@utility glass {
  border-radius: 1.5rem;
  background: rgb(255 255 255 / 0.10);
  backdrop-filter: blur(22px) saturate(1.2);
  -webkit-backdrop-filter: blur(22px) saturate(1.2);
  border: 1px solid rgb(255 255 255 / 0.30);
  box-shadow: 0 24px 60px rgb(0 0 0 / 0.18), inset 0 1px 0 rgb(255 255 255 / 0.35);
  color: #fff;
}
```
```html
<div class="glass p-6">Frosted card</div>
```

## Option C — plugin (Tailwind v3)

```js
// tailwind.config.js
const plugin = require("tailwindcss/plugin");
module.exports = {
  plugins: [
    plugin(({ addComponents }) => {
      addComponents({
        ".glass": {
          borderRadius: "1.5rem",
          background: "rgb(255 255 255 / 0.10)",
          backdropFilter: "blur(22px) saturate(1.2)",
          "-webkit-backdrop-filter": "blur(22px) saturate(1.2)",
          border: "1px solid rgb(255 255 255 / 0.30)",
          boxShadow: "0 24px 60px rgb(0 0 0 / 0.18), inset 0 1px 0 rgb(255 255 255 / 0.35)",
          color: "#fff",
        },
      });
    }),
  ],
};
```

## Fallbacks in Tailwind
```html
<div class="glass supports-[not_(backdrop-filter:blur(0))]:bg-slate-900/85
            motion-reduce:backdrop-blur-none">…</div>
```
Or put the `@supports` / `prefers-reduced-transparency` blocks (see `../SKILL.md`) in your global CSS once.
