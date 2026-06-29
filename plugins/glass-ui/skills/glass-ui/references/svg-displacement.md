# Glass UI — SVG Displacement Refraction (the middle tier)

Between cheap CSS blur and a full WebGL shader sits a third technique: **SVG filters**.
`feDisplacementMap` warps an element's pixels by an offset stored in another image's
color channels — i.e. real **refraction** (light-bending at the edges), with **no canvas
and no WebGL**, just a `<filter>` and a CSS `backdrop-filter: url(#id)`.

Use this when you want the background to *bend* through the glass (not merely blur), but
don't want to manage a WebGL context. It's the sweet spot for web "liquid glass".

## How `feDisplacementMap` works

```
feDisplacementMap(in=SourceGraphic, in2=MAP, scale=S, xChannelSelector=R, yChannelSelector=G)
```
For each pixel it reads a color from `MAP`, turns the chosen channels into a vector, and
**moves the source pixel by that vector × scale**. So `MAP` *is* the lens: wherever the map
is mid-gray (0.5, 0.5) → no shift; brighter/darker R or G → push right/left or up/down.
Put the strong values near the rounded edges and you get edge refraction with a calm center —
exactly the squircle-lens look.

## A. Pure CSS/SVG frosted distortion (no JS) — quickest

Good for a wobbly, frosted-glass distortion. Uses `feTurbulence` to generate the map:

```html
<svg width="0" height="0" style="position:absolute">
  <filter id="glass-distort" x="0" y="0" width="100%" height="100%">
    <feTurbulence type="fractalNoise" baseFrequency="0.008 0.008"
                  numOctaves="2" seed="4" result="noise"/>
    <feGaussianBlur in="noise" stdDeviation="2" result="smooth"/>
    <feDisplacementMap in="SourceGraphic" in2="smooth" scale="60"
                       xChannelSelector="R" yChannelSelector="G"/>
  </filter>
</svg>
```
```css
.glass-refract {
  backdrop-filter: blur(2px) url(#glass-distort) saturate(1.3);
  -webkit-backdrop-filter: blur(2px) saturate(1.3); /* Safari ignores url() filters on backdrop */
  background: rgb(255 255 255 / 0.10);
  border: 1px solid rgb(255 255 255 / 0.30);
  border-radius: 24px;
  box-shadow: 0 24px 60px rgb(0 0 0 / 0.18), inset 0 1px 0 rgb(255 255 255 / 0.35);
}
```
- `scale` = refraction strength. `baseFrequency` low = large smooth ripples; high = busy frost.
- This is a *noisy* distortion, not a precise lens. For a clean magnifying lens, use B.

## B. Precise edge-lens (compute the displacement map) — the "real" liquid glass

The clean look (shuding/liquid-glass uses this) feeds `feDisplacementMap` a **computed**
map via `feImage` instead of noise. You generate the map once on a hidden `<canvas>`:

For each pixel of the tile, compute its **signed distance to the rounded-rect edge**, turn
that into a displacement that is ~0 in the interior and ramps up near the rim
(`smoothstep`), point it toward the center (so edges magnify), then **encode** the (dx, dy)
offset into the R and G channels (`channel = offset / maxScale + 0.5`, so 0.5 = no shift).
Set `feImage href` to `canvas.toDataURL()` and `feDisplacementMap scale = maxScale`.

Pseudocode for the map generator:
```js
// for each pixel (px,py) inside the tile:
const d   = sdRoundedRect(px, py, halfW, halfH, radius);   // <0 inside, 0 at edge
const amt = smoothstep(0.0, edgeWidth, -d);                // 0 center → 1 near rim... invert to taste
const dir = normalize(center - pixel);                     // push toward center = magnify
const dx  = dir.x * amt, dy = dir.y * amt;
R = dx / maxScale + 0.5;   // encode (0.5 == no displacement)
G = dy / maxScale + 0.5;
```
Then the filter is just:
```html
<filter id="lens">
  <feImage href="<dataURL of the map>" result="map"/>
  <feDisplacementMap in="SourceGraphic" in2="map" scale="<maxScale>"
                     xChannelSelector="R" yChannelSelector="G"/>
</filter>
```
Recompute the map only when the tile **resizes** (not every frame) — that's why this is far
cheaper than the per-frame WebGL shader.

## Tradeoffs vs the other tiers

| | CSS blur | **SVG displacement** | WebGL shader |
|---|---|---|---|
| Refraction (bends bg) | ❌ blur only | ✅ yes | ✅ yes |
| Needs canvas/WebGL | no | no (tiny hidden canvas only for map in B) | yes |
| Cost | cheapest | cheap (map computed on resize) | highest (per-frame) |
| Safari support | ✅ | ⚠️ `url()` on `backdrop-filter` is flaky in Safari — test & fall back to blur | ✅ |
| Best for | panels, navbars | hero cards, "liquid" tiles | showpieces, draggable lenses, exports |

## Gotchas
- **Safari**: `backdrop-filter: url(#f)` is unreliable; many ship a `@supports`/UA fallback to
  plain `blur()`. Always provide the blur fallback (the `-webkit-` line above omits the url()).
- `feDisplacementMap` on a **backdrop** is newer than on the element's own content; if a browser
  won't displace the backdrop, apply the filter to a background layer element instead.
- Keep `scale` modest (20–80). Too high looks like a funhouse mirror, not glass.
- Study `_reference/shadcn-registries/shuding-liquid-glass/liquid-glass.js` for a complete
  console-paste implementation of approach B.
