---
name: glass-gl
description: >-
  Add real WebGL "liquid glass" to a web UI with the glass-gl engine — any DOM
  element refracts a background (image / canvas / video) like a physical glass
  lens: bend, magnify, frost, edge-light, and chromatic edge fringe, not just a
  backdrop-filter blur. Use when the user wants Apple-style "liquid glass",
  refraction / lens effects, glass cards/panels/tiles over a photo or video, or
  asks for glass-gl by name. Do NOT use for frosted glass over live scrolling
  page content — that's CSS backdrop-filter, not this.
---

# glass-gl

`glass-gl` is a tiny, framework-agnostic **WebGL liquid-glass engine**. It runs one
fullscreen `<canvas>` behind the page and draws a refracting lens at each element you
register, every frame — so the glass tracks elements as they move, drag, or resize.

- npm: **`glass-gl`** · repo: **https://github.com/wiiiimm/glass-gl** · MIT · ESM, zero deps
- Status: **experimental** (API/visuals may change)

## The one rule (read this first)

**It bends a *background*, not the *page*.** The shader only samples the background
texture you give it (an image, a `<canvas>`, or a `<video>`). It **cannot** refract
arbitrary live DOM sitting behind it (e.g. a `<p>` or a scrolling article).

- Put your content **on** the glass (the element's own children sit on top, crisp).
- Give the glass a **media background** to refract (photo / gradient / video / canvas).
- To make the glass bend **text or graphics**, **bake them into the background canvas**
  (draw the text into the `<canvas>` you pass as the background) — then the lens refracts
  them too. This is the only way to get "glass over text" with this technique.

If what the user actually wants is a frosted panel over live, scrolling page content,
the right tool is CSS `backdrop-filter: blur()` — not glass-gl. Say so.

| Use glass-gl when… | Use CSS `backdrop-filter` when… |
|---|---|
| Glass over a photo / video / gradient / canvas | Frosted panel over live scrolling DOM |
| You want real **refraction** (the bg bends/magnifies) | You only need a blur/tint |
| Draggable lenses, hero sections, media backdrops | Sticky navbars, modals over content |

## Install

```bash
npm i glass-gl
```

```js
import { createGlass } from "glass-gl";
```

No build step? Import straight from a CDN:

```js
import { createGlass } from "https://esm.sh/glass-gl";
```

## Quickstart

```html
<canvas id="gl" style="position:fixed; inset:0; z-index:-1"></canvas>
<div class="card" style="position:fixed; left:80px; top:120px; width:320px; height:200px;
     border-radius:24px; padding:24px; color:#fff">Liquid glass</div>

<script type="module">
  import { createGlass } from "https://esm.sh/glass-gl";

  const glass = createGlass({
    canvas: document.getElementById("gl"),
    background: "/photo.jpg",        // the thing the glass refracts
  });

  glass.register(document.querySelector(".card"), { radius: 24 }); // match border-radius
</script>
```

The card now refracts `/photo.jpg`. Its text stays sharp on top; the glass bends only the
background behind it.

## API

`createGlass({ canvas, background, params, dpr, transparent }) → instance`

### Options

| Option | Default | Meaning |
|---|---|---|
| `dpr` | on (≤2) | Retina sharpness: render at `devicePixelRatio`, clamped to 2. Pass a number for a custom cap, `false`/`0` for legacy 1:1. |
| `transparent` | `false` | Draw **only** the glass surfaces; every other pixel stays transparent (premultiplied alpha). Use when the background you refract **is the page's own live canvas** (see recipe below). Default mode paints the background across the whole canvas. |

| Method | What it does |
|---|---|
| `register(el, { radius })` | Make `el` a glass surface. Pass `radius` = the element's `border-radius` (use `height/2` for a pill, `size/2` for a circle). Returns an unregister fn. |
| `unregister(el)` | Stop treating `el` as glass. |
| `clear()` | Unregister everything. |
| `setParams({ … })` | Update look (see params below). Merges. |
| `getParams()` | Current params. |
| `setBackground(src)` | Swap what gets refracted. `src` = URL string, `<img>`, `<canvas>`, or `<video>`. |
| `destroy()` | Stop the render loop and release. |

### Params (`setParams` / `createGlass({ params })`)

| Param | Range | Meaning |
|---|---|---|
| `refraction` | ~0–0.5 | Lens strength — how much the background bends/magnifies. |
| `blur` | px | Frost (sample spread). |
| `liquidness` | 0–~0.6 | Milky mix toward `tint`. |
| `edgeLight` | ~0–2 | Strength of the **directional rim glint** (bright arc facing the light, soft shade opposite). |
| `edgeFrost` | 0–1 | Rim band (a frosted border). `0` = none (since 0.3.0 — older versions kept a faint rim even at 0). |
| `dispersion` | 0–1 | **Chromatic aberration** — splits R/B at the rim so white edges fringe into colour like real glass. Off by default. |
| `saturation` | ~0.8–1.7 | **Vibrancy** — saturates the refracted backdrop (Apple-material style). `1` = neutral, ~`1.2–1.35` = luminous. |
| `curve` | 1–4 | **Lens profile** — `1` = linear bend; `~2.5–3.5` = droplet (optically flat centre, steep bend only at the rim — the "liquid" look). |
| `lightAngle` | deg | Direction the rim glint comes from (`0` = top, `90` = right). |
| `radius` | px | Default corner radius (override per element via `register`). |
| `tint` | `[r,g,b]` 0–1 | Glass colour (`[1,1,1]` = clear/light; dark values = dark glass). |

## Live backgrounds (canvas & video)

A `<canvas>` or `<video>` passed to `setBackground` is treated as **live** — re-uploaded
every frame — so animated backgrounds refract in real time.

```js
// Playing video, refracted live:
const v = Object.assign(document.createElement("video"),
  { src: "/clip.mp4", loop: true, muted: true, autoplay: true, playsInline: true });
v.play();
glass.setBackground(v);

// Or paint your own canvas each frame (Ken Burns, generative art, baked text…):
const stage = document.createElement("canvas");
const ctx = stage.getContext("2d");
function draw() {
  ctx.drawImage(photo, panX, panY, w, h);     // your motion
  ctx.fillStyle = "#fff"; ctx.font = "700 120px serif";
  ctx.fillText("glass-gl", cx, cy);            // baked text → the glass refracts it
  requestAnimationFrame(draw);
}
draw();
glass.setBackground(stage);
```

## Glass over the page's OWN canvas (transparent mode)

When the page already renders its background itself — a three.js scene, a game, a map,
any live `<canvas>` — don't let the engine repaint it (the fullscreen copy re-samples
and softens the whole page). Pass that canvas as the background **and** set
`transparent: true`: the page's canvas stays visible and crisp, and the engine draws
only the lenses on top.

```html
<canvas id="scene"></canvas>                                  <!-- your renderer -->
<canvas id="glass" style="position:fixed; inset:0; pointer-events:none"></canvas>
```

```js
const glass = createGlass({
  canvas: document.getElementById("glass"),
  background: document.getElementById("scene"),   // live: re-uploaded every frame
  transparent: true,                              // draw ONLY the glass
});
glass.register(panel, { radius: 20 });
```

Stack the glass canvas **above** the scene canvas and below your DOM content. This is
the recommended setup for HUD/panel glass over WebGL scenes.

## Gotchas

- **Needs a busy/colourful background** to be visible — never put it over a flat colour.
- **Match `radius` to the element's `border-radius`**, or the corners mismatch.
- **External images need CORS** (`crossOrigin="anonymous"` + the host sending
  `Access-Control-Allow-Origin`), or the WebGL texture upload fails. Same-origin/bundled
  images are simplest.
- **It runs a continuous `requestAnimationFrame` loop** (GPU stays awake). Fine for a
  desktop showpiece; for mobile/always-on, consider pausing when offscreen.
- **Multiple surfaces:** up to 16 registered glass surfaces at once (shader array size).

## More

- The quickstart above is the minimal runnable demo — paste it into an `.html` file.
- Full interactive playground (drag glass cards, tune every param live) and the
  source of truth for the engine: https://github.com/wiiiimm/glass-gl
