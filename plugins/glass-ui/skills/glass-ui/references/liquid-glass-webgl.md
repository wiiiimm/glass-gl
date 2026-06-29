# Glass UI — WebGL Liquid Glass (advanced)

`backdrop-filter` only blurs. A *physical* glass tile also **refracts** — it bends
and magnifies the background through its curved edges. That requires sampling the
background per-pixel in a fragment shader. Use only for showpieces; gate behind
`prefers-reduced-motion` with the CSS-glass version as fallback.

## How it works (the four ideas)

A full-screen `<canvas>` sits *behind* your DOM and renders the page background as a
texture. A fragment shader runs once per screen pixel and, for pixels inside the
glass tile's rectangle, does:

1. **Squircle mask.** A rounded rectangle via a superellipse:
   `box = pow(|dx|, n) + pow(|dy|, n)` where `dx,dy` are the pixel's distance from the
   tile center divided by the tile half-size. `n = 6` gives Apple-style rounded
   corners. `box < 1` is inside. Soft-threshold it for anti-aliased edges and to carve
   a bright rim band.
2. **Lens magnification.** Pull the sample coordinate toward the tile center based on
   how close to the edge you are: `sampleUV = center + (uv - center) * (1 - box * k)`.
   `k ≈ 0.22`. This is what bends the background — stronger near the edges, like a real
   lens.
3. **Frost (blur).** Average an N×N grid of texture samples around `sampleUV` (e.g. 9×9).
   Bigger grid = frostier and slower.
4. **Edge lighting.** Add a vertical brightness gradient + the rim band so the top edge
   catches light. Optionally `mix()` toward white for a "milky"/liquidness amount.

Feed the shader the tile's live screen rect each frame, so dragging/resizing the tile
makes the refraction track it.

## Distilled fragment shader (WebGL1 / GLSL ES 1.0)

```glsl
precision mediump float;
uniform vec3  iResolution;  // canvas px
uniform vec2  uTileCenter;  // tile center in px, y-up
uniform vec2  uTileHalf;    // tile half-size in px
uniform float uMilk;        // 0..1 push toward white
uniform sampler2D uBg;      // background texture

void main() {
  vec2 frag = gl_FragCoord.xy;
  vec2 uv   = frag / iResolution.xy;
  vec2 d    = (frag - uTileCenter) / uTileHalf;

  float box  = pow(abs(d.x), 6.0) + pow(abs(d.y), 6.0);   // squircle field
  float body = clamp((1.0 - box) * 8.0, 0.0, 1.0);        // soft inside mask
  float rim  = clamp((0.96 - box) * 16.0, 0.0, 1.0)
             - clamp((0.90 - box) * 16.0, 0.0, 1.0);       // bright edge band

  vec4 bg = texture2D(uBg, uv);
  vec4 col = bg;

  float inside = clamp(body + rim, 0.0, 1.0);
  if (inside > 0.0) {
    vec2 c    = uTileCenter / iResolution.xy;
    vec2 lens = c + (uv - c) * (1.0 - box * 0.22);          // refraction

    vec4 acc = vec4(0.0);                                    // 9x9 frost blur
    for (float x = -4.0; x <= 4.0; x++)
      for (float y = -4.0; y <= 4.0; y++)
        acc += texture2D(uBg, lens + vec2(x, y) * 1.2 / iResolution.xy);
    acc /= 81.0;

    float light = clamp(uv.y - c.y, 0.0, 0.2) + 0.1;        // top-edge highlight
    vec4 glass  = clamp(acc + body * light + rim * 0.3, 0.0, 1.0);
    glass = mix(glass, vec4(1.0), uMilk * 0.95);
    col   = mix(bg, glass, inside);
  }
  gl_FragColor = vec4(col.rgb, 1.0);
}
```

## Wiring (sketch)
1. Full-screen quad (`TRIANGLE_STRIP`, 4 verts), trivial vertex shader passing position.
2. Upload the background image with `texImage2D`; set `LINEAR` filtering, `CLAMP_TO_EDGE`,
   `UNPACK_FLIP_Y_WEBGL = true` (WebGL y-up vs DOM y-down).
3. Each frame: read your DOM tile's `getBoundingClientRect()`, convert to canvas px with
   y flipped (`cy = canvas.height - (rect.top + rect.height/2)`), set the uniforms,
   `drawArrays`. Render your real DOM content *on top* of the canvas (transparent tile bg).
4. Background must be **CORS-readable** if you ever want to export the canvas to PNG.

## When NOT to use this
- Static cards, navbars, modals → CSS `backdrop-filter` is 1/100th the code and GPU.
- Long lists of glass items → per-pixel cost scales with screen area; one hero tile is fine.
