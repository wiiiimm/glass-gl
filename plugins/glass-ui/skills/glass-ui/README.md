# Glass UI

A small, portable **skill** for building glassmorphism and "liquid glass" interfaces —
frosted translucent cards, navbars, buttons, modals — in whatever stack you use.

![what it produces](#) <!-- open assets/example.html to see it live -->

## What you get

- A battle-tested **CSS recipe** for frosted glass (the four-layer formula).
- Ready-to-paste code for **plain CSS, SCSS, Tailwind, and shadcn/React**.
- An advanced **WebGL liquid-glass shader** for true light-bending refraction.
- A drop-in stylesheet (`assets/glass.css`) and a live demo (`assets/example.html`).
- Built-in **accessibility & performance** guidance (fallbacks, reduced transparency).

## See it first

Open the demo in a browser:

```bash
cd assets && python3 -m http.server 8000
# then visit http://localhost:8000/example.html
```

You'll see a frosted nav bar, a light-glass card, and a dark-glass card over a
colorful gradient — all from `glass.css`, no images required.

## Use it in 30 seconds (any HTML/CSS project)

1. Copy `assets/glass.css` into your project and link it.
2. Make sure there's something colorful behind your element (a photo or gradient) —
   **glass over a flat color is invisible.**
3. Add a class:

```html
<div class="glass" style="padding:24px">Frosted</div>
<button class="glass-btn">Button</button>
<input class="glass-input" placeholder="Search" />
<nav class="glass-nav">…</nav>
```

Tune it with CSS variables (`--glass-blur`, `--glass-fill`, `--glass-tint`, …).

## Use it in your framework

| Stack | File |
|-------|------|
| Plain CSS | [`references/css.md`](references/css.md) |
| SCSS | [`references/scss.md`](references/scss.md) |
| Tailwind | [`references/tailwind.md`](references/tailwind.md) |
| shadcn / React | [`references/shadcn.md`](references/shadcn.md) |
| SVG displacement (refraction, no WebGL) | [`references/svg-displacement.md`](references/svg-displacement.md) |
| WebGL liquid glass | [`references/liquid-glass-webgl.md`](references/liquid-glass-webgl.md) |

## The one thing to remember

Glass = **translucent fill + `backdrop-filter: blur()` + bright thin border +
drop shadow + inset top highlight**, placed over a busy background. Everything else
is tuning. See [`SKILL.md`](SKILL.md) for the full recipe and the rules that make or
break the effect.

## How this is used as an AI skill

`SKILL.md` carries YAML frontmatter (`name`, `description`). Agent tools that support
skills (Claude Code, and other skill-aware assistants) read the description to decide
when to apply it, then load the references on demand. You can also just read the files
yourself — it's plain Markdown and CSS.

---
*Distilled from studying a production liquid-glass card; the CSS recipe is standard
glassmorphism and the shader is an original, annotated reimplementation.*
