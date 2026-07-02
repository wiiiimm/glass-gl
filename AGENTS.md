# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Codex, etc.) working in this repo.
`CLAUDE.md` is a symlink to this file — keep all agent guidance here.

## What this project is

**`glass-gl`** — a tiny, framework-agnostic **WebGL "liquid glass" engine**. It makes any DOM
element refract the background behind it like a physical glass lens (bend, magnify, frost,
edge-light), not just a `backdrop-filter` blur. Status: **experimental**.

The repo ships:
- **The engine** — `packages/glass-gl/` (the npm package, name `glass-gl`).
- **Interactive demos** — `playground/` — `index.html` (drag glass cards over a photo, tune every
  parameter live) and `os.html` (**glassOS**, a responsive desktop made entirely of glass). Both
  are *consumers* of the engine (they prove the library works).

> Naming is settled: the product/package is **`glass-gl`** (repo `wiiiimm/glass-gl`). The local
> folder is still named `glass-ui-kit` for historical reasons — that's fine, it doesn't matter.

## The core idea (and its hard limits)

- **It's JS + WebGL, not CSS.** The glass is a fragment shader on a `<canvas>` behind the page.
  You can't get this refraction with CSS alone — CSS can only blur (`backdrop-filter`).
- **It bends a *background*, not the *page*.** `setBackground` takes a URL string, `<img>`,
  `<canvas>`, or `<video>` (for a gradient, paint it into a canvas and pass that). It **cannot**
  refract arbitrary live DOM behind it
  (e.g. a `<p>`). Put content *on* the glass; give the glass a media background to refract. To
  refract text/graphics, **bake them into the background canvas** (that's what the playground does
  with the `glass-gl` wordmark). A `<canvas>`/`<video>` background is **live** — re-uploaded every
  frame — so Ken Burns slideshows and playing video refract in real time.
- **The "liquid" look is three shader ingredients** (all params, see `skills/glass-gl/SKILL.md`):
  `curve` (droplet lens profile — flat centre, steep rim), `dispersion` (chromatic R/B fringe at
  the rim), and `saturation` (vibrancy of the refracted backdrop). `edgeLight` + `lightAngle`
  drive a directional specular rim glint (bright arc facing the light, shade opposite).
- For "frosted panel over live scrolling content", CSS `backdrop-filter` is the right tool — not
  this. `glass-gl` is for glass over a media background: heroes, photo/video backdrops, tiles.
- **Resource note:** the engine runs a continuous `requestAnimationFrame` loop (GPU stays awake).
  Fine for a desktop showpiece; for mobile/always-on, an idle-pause would be worth adding.

## Engine = one file (no drift)

`packages/glass-gl/glass-gl.js` is the **canonical source**. `playground/glass-gl.js` is a
**symlink** to it (git mode `120000`). **Edit the package file**; never replace the symlink with a
copy. npm still ships the real file; the demo follows the symlink, so they can't diverge.

Public API: `createGlass({ canvas, background, dpr, transparent })` → `register(el, { radius })` /
`unregister` / `clear` / `setParams({...})` / `setBackground(src)` / `destroy()`. `setBackground`
takes a URL string, `<img>`, `<canvas>`, or `<video>` (canvas/video = live, refreshed each frame).
Options: `dpr` = retina-sharp buffers by default (≤2×; number to cap, `false` for 1:1);
`transparent: true` = draw only the lenses (premultiplied alpha) for glass over the page's own
live canvas (demo: `playground/index.html?transparent`). Params include `dispersion`,
`saturation`, `curve`, and `lightAngle` alongside `refraction`/`blur`/`edgeFrost`/etc.
(`edgeFrost: 0` = no rim since 0.3.0) — `skills/glass-gl/SKILL.md` has the full table.

## Repository layout

- `packages/glass-gl/` — the npm package: `glass-gl.js` (canonical engine), `package.json`, `README.md`.
- `playground/` — interactive demos: `index.html` (parameter playground), `os.html` (glassOS — a
  responsive glass desktop: menu bar, dock, app windows as info pages), `glass-gl.js` (symlink),
  `images/`. **Wired to a Vercel deployment — don't restructure this directory.**
- `skills/glass-gl/SKILL.md` — the distributable agent skill (installable via `npx skills add
  wiiiimm/glass-gl`). **Canonical agent-facing usage doc** — when the library's API/usage changes,
  update `SKILL.md` (don't re-document it in a new place). It references the engine by npm/CDN, never
  a copy — no engine duplication.
- `.github/workflows/publish.yml` — manual-trigger npm publish + GitHub Release.
- `.claude/skills/` — installed release/commit skills (local dev reference only — NOT distributed).
- `docs/` — screenshots used in the READMEs.
- `AGENTS.md` / `CLAUDE.md` (symlink), `README.md`, `LICENSE`.

## Release & commit workflow

**Current method — kept deliberately simple while experimental:**

- **Use Conventional Commits** — `type(scope): description` (`feat`/`fix`/`chore`/`ci`/`docs`/…;
  `feat!` or a `BREAKING CHANGE:` footer for majors). Free good habit; makes a later switch to
  semantic-release trivial.
- **Releasing = bump version, then run the workflow (manual, not on push).** Edit the `version`
  in `packages/glass-gl/package.json` and commit. Then trigger the **Publish glass-gl** workflow —
  Actions tab → *Run workflow*, or `gh workflow run "Publish glass-gl" --repo wiiiimm/glass-gl`.
  It publishes to npm + cuts a GitHub Release, but **only when that version isn't already on npm**
  (a stray run won't republish), and it does **not** run on push (no surprise-publish).
- `glass-gl` is a **package**, so `npm publish` *is* the release — there is no prod deploy to gate.
- One-time setup: an **`NPM_TOKEN`** repo secret is required for CI to publish (see README / repo
  settings). `v0.0.1` was the first publish (done via the npm CLI before CI existed).

**Deferred (until there are users / a real cadence):** full semantic-release automation,
commit-driven auto-versioning, changelog generation, pooled-release triggers. Reference skills for
that day live in `.claude/skills/` (`conventional-commits`, `semantic-release-automation`,
`pooled-release`, `production-release-gating`).

**`production` branch** exists but is **reserved for the future demo-site deploy** (Vercel gating —
"don't deploy every commit to prod"), NOT for the npm package.

## Conventions

- **Glass needs a busy/colorful background to be visible** — every demo places glass over a photo
  or gradient, never a flat color.
- **Content goes *on* the glass**, not behind it (the shader can't refract DOM — see limits above).
- **Match each element's `border-radius`** to its registered `radius` (the engine takes a
  per-surface radius), or corners mismatch.
- Glass is **visual — look, don't just read.** Verify rendering, don't assume.

## How to verify changes (render it)

```bash
# serve the playground
python3 -m http.server 8000 --directory playground   # open http://localhost:8000

# or capture a headless screenshot (renders WebGL via SwiftShader):
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new \
  --use-gl=angle --use-angle=swiftshader --window-size=1440,900 --virtual-time-budget=5000 \
  --screenshot=/tmp/glass.png "http://localhost:8000/index.html"
```
Check: cards refract (bend) the background, edges are crisp with the rim, gloss sheen sits over
content, panel/footer render as glass, text stays legible.

## Reference material (outside this repo)

Early research + downloaded peer projects live at `../glass-ui/_reference/` (`NOTES.md` has the
competitive comparison); the original studied liquid-glass card is at `../glass-ui/bubbbly/`.
Study and reimplement — do not copy peer code into this repo.
