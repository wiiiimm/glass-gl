# glass-gl

Real **WebGL liquid glass** for the web — a tiny, framework-agnostic engine that makes
any DOM element refract the background behind it like a physical glass lens (bend +
magnify + frost + edge light), not just a `backdrop-filter` blur.

![glass-gl playground — draggable glass cards refracting a photo background, with a live control panel](docs/playground.jpg)

> **⚠️ Experimental.** Early and evolving — the API, parameters, and visuals may change
> without notice, and it isn't production-hardened yet. The engine works; packaging and a
> React wrapper are in progress.

## What it does

You give it a **background to refract** (image, canvas, video, gradient) and register the
elements you want to be glass. The engine runs one fullscreen WebGL canvas behind your
page and draws a refracting lens at each registered element's position every frame — so
the glass tracks elements as they move, drag, or resize.

A `<canvas>` or `<video>` background is treated as **live** — re-uploaded every frame — so
animated backgrounds (a playing video, a Ken Burns slideshow, anything you paint into a
canvas) refract in real time. Want the glass to bend *text or graphics*? Draw them into
that background canvas (the lens only refracts the background layer — see the rule below).

```js
import { createGlass } from "glass-gl";

const glass = createGlass({ canvas, background: "/bg.jpg" });

glass.register(el);          // any element becomes liquid glass
glass.setParams({
  refraction: 0.22,          // lens strength
  blur: 1.2,                 // frost
  liquidness: 0.0,           // milky-white mix
  edgeLight: 1.0,            // top sheen
  edgeFrost: 0.22,           // rim border
  dispersion: 0.2,           // chromatic aberration at the edge (white → RGB fringe)
  saturation: 1.25,          // vibrancy — saturate the refracted backdrop
  curve: 2.8,                // lens profile: 1 linear → ~3 droplet (flat centre, steep rim)
  lightAngle: 35,            // where the rim glint comes from (deg, 0 = top)
  radius: 30,                // match the element's border-radius
  tint: [1, 1, 1],           // glass colour
});

glass.unregister(el);
glass.setBackground("/other.jpg");   // string / <img> = static
glass.setBackground(videoEl);        // <video> or <canvas> = LIVE, refracted every frame
glass.destroy();
```

`createGlass` options: **`dpr`** (retina-sharp by default, clamped to 2× — pass a number to
cap it or `false` for 1:1) and **`transparent: true`** (draw *only* the lenses, transparent
elsewhere — for glass over the page's **own** live canvas, e.g. a three.js scene, so the
page stays crisp). Since 0.3.0, `edgeFrost: 0` means **no** rim band.

## The one rule

Liquid glass needs **something to bend**. The effect refracts a *background layer you give
it* — a photo, video, gradient, or canvas. It does **not** refract arbitrary live DOM
sitting behind it (that would require rasterising the page every frame). For "frosted glass
over scrolling content," the browser's own `backdrop-filter` is the right tool; `glass-gl`
is for glass over a media/background surface — hero sections, backdrops, draggable tiles.

## Use it as an agent skill

Coding agents (Claude Code, Cursor, Copilot CLI, …) can install a glass-gl skill that teaches
them how to add the effect correctly:

```bash
npx skills add wiiiimm/glass-gl
```

The skill lives at [`skills/glass-gl/SKILL.md`](skills/glass-gl/SKILL.md) — the single,
canonical "how to use glass-gl" reference (the one rule, API, params, live backgrounds,
gotchas). It references the engine by npm/CDN, so there's no second copy to drift.

## Demo

The `playground/` directory ships two demos, both plain consumers of the same `glass-gl.js`
you'd install:

- **`index.html` — glassOS** (the home page). A tiny responsive desktop where *everything*
  is glass: menu bar, dock, draggable app windows. The windows are pages — About, How it
  works, Skills & install (with "For humans / For AI agents" tabs), Wallpapers, Settings,
  and a Terminal. The dock links out to the playground.
- **`playground.html` — the parameter playground.** Drag glass cards over a photo and tune
  every parameter live (presets, sliders, copy-as-code). Add `?transparent` to see
  transparent mode over the page's own live canvas.

```bash
cd playground && python3 -m http.server 8000
# open http://localhost:8000                  (glassOS)
# open http://localhost:8000/playground.html  (parameter playground)
```

## Releasing

`glass-gl` ships from `packages/glass-gl`, and the release process is kept simple while the
project is experimental:

1. Bump the `version` in `packages/glass-gl/package.json` and commit (Conventional Commits —
   `feat:` / `fix:` / `chore:` …).
2. Run the **Publish glass-gl** workflow — GitHub → Actions → *Run workflow*, or
   `gh workflow run "Publish glass-gl"`.
3. CI publishes to npm and cuts a matching GitHub Release — only when that version isn't already
   on npm. It does **not** run on push, so a routine commit can't surprise-publish.

> The engine lives once: `packages/glass-gl/glass-gl.js` is canonical and `playground/glass-gl.js`
> is a symlink to it — edit the package file.

Full semantic-release automation and the `production` deploy gate are intentionally deferred;
see `AGENTS.md` and `.claude/skills/` for that setup when it's needed.

## License

AGPL-3.0-or-later © wiiiimm — see [LICENSE](LICENSE). If you run a modified version
as part of a network service, the AGPL requires you to offer its source to users.
(Versions ≤ 0.0.2 were published under MIT and remain so.)
