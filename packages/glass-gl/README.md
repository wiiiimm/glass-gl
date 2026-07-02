# glass-gl

Real **WebGL liquid glass** for the web. A tiny, framework-agnostic engine that makes any
DOM element refract the background behind it like a physical glass lens — bend, magnify,
frost, edge-light — not just a `backdrop-filter` blur.

![glassOS — a desktop made of glass-gl surfaces: menu bar, dock, and draggable app windows refracting a floral wallpaper with the glassOS wordmark baked in](https://raw.githubusercontent.com/wiiiimm/glass-gl/main/docs/playground.jpg)

> **⚠️ Experimental.** Early and evolving — the API, parameters, and visuals may change
> without notice, and it is not production-hardened. Use at your own risk.

## Install

```bash
npm install glass-gl
```

## Use

```js
import { createGlass } from "glass-gl";

const glass = createGlass({ canvas, background: "/bg.jpg" });

glass.register(el);          // any element becomes liquid glass
glass.setParams({
  refraction: 0.22,          // lens strength
  blur: 1.2,                 // frost
  liquidness: 0.0,           // milky mix toward tint
  edgeLight: 1.0,            // directional rim glint strength
  edgeFrost: 0.22,           // frosted rim band (0 = none)
  dispersion: 0.2,           // chromatic aberration at the rim
  saturation: 1.25,          // vibrancy of the refracted backdrop
  curve: 2.8,                // lens profile: 1 linear → ~3 droplet
  lightAngle: 35,            // rim-glint light direction (deg, 0 = top)
  radius: 30,                // match the element's border-radius
  tint: [1, 1, 1],
});

glass.unregister(el);
glass.setBackground("/other.jpg");   // string / <img> = static; <canvas> / <video> = live
glass.destroy();
```

### Options (`createGlass({ … })`)

- **`dpr`** — retina-sharp by default: the canvas renders at `devicePixelRatio`
  (clamped to 2). Pass a number for a custom cap, or `false`/`0` for legacy 1:1.
- **`transparent: true`** — draw **only** the glass surfaces and leave every other
  pixel transparent (premultiplied alpha). Use it when the background you refract
  **is the page's own live canvas** (a three.js scene, a game, a visualisation):
  the page's pixels stay crisp and the engine composites just the lenses on top.
  Default (opaque) mode paints the background across the whole canvas — right for
  glass over a media backdrop you hand to the engine.

```js
// glass over YOUR live canvas (e.g. three.js) — no background re-display:
const glass = createGlass({ canvas: glassCanvas, background: sceneCanvas, transparent: true });
```

> **Behaviour change in 0.3.0:** `edgeFrost: 0` now means *no* rim band. Earlier
> versions kept a faint hard-coded rim even at 0 — scenes relying on it should set
> `edgeFrost: ~0.17`.

## When to use it (and when not to)

`glass-gl` is **JavaScript + WebGL, not CSS**. It renders the glass on a `<canvas>` behind
your page and refracts a **background layer you give it**. That buys real light-bending
(not just blur) — but it comes with one hard rule.

### The one rule: it bends a *background*, not the *page*

The shader only samples the background texture you provide. It has **no access to your DOM**,
so it cannot refract other HTML elements sitting behind it.

| Arrangement | Works? |
|---|---|
| Content (text, images) **on top of** a glass surface | ✅ yes |
| Glass refracting a **background** image / canvas / video / gradient | ✅ yes |
| Glass refracting a **live DOM element behind it** (e.g. a `<p>` of text) | ❌ no — it refracts the background, not the element |

Bending live DOM would mean rasterising the page every frame (slow, CORS-bound) — which is
why even Apple's Liquid Glass runs in the OS compositor, not the browser.

### Reach for CSS instead when…

If you want a **frosted panel over live, scrolling content** (text, a feed, a list), use the
browser's own `backdrop-filter: blur()` — it blurs real DOM behind it, live and for free. It
just can't *refract*. Use `glass-gl` for glass **over a media background**: hero sections,
photo/video backdrops, draggable tiles, showpieces.

## Using an AI coding agent?

Install the glass-gl skill so your agent knows how to wire it up correctly:

```bash
npx skills add wiiiimm/glass-gl
```

## License

AGPL-3.0-or-later © wiiiimm · https://github.com/wiiiimm/glass-gl — see the bundled
LICENSE. If you run a modified version as part of a network service, the AGPL requires
you to offer its source to users. (Versions ≤ 0.0.2 were published under MIT.)
