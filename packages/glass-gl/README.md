# glass-gl

Real **WebGL liquid glass** for the web. A tiny, framework-agnostic engine that makes any
DOM element refract the background behind it like a physical glass lens — bend, magnify,
frost, edge-light — not just a `backdrop-filter` blur.

![glass-gl playground — glass cards refracting a photo background, with the glass-gl wordmark baked into the scene and a live control panel](https://raw.githubusercontent.com/wiiiimm/glass-gl/main/docs/playground.jpg)

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
  edgeFrost: 0.22,           // frosted rim band
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

MIT © wiiiimm · https://github.com/wiiiimm/glass-gl
