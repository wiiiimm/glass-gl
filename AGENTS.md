# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Codex, etc.) working in this repo.
`CLAUDE.md` is a symlink to this file — keep all agent guidance here.

## Release & commit workflow

**Current method — kept deliberately simple while the project is experimental:**

- **Use Conventional Commits** — `type(scope): description` (`feat`/`fix`/`chore`/`ci`/`docs`/…;
  `feat!` or a `BREAKING CHANGE:` footer for majors). Free good habit, and makes a later switch
  to semantic-release trivial.
- **Releasing = manual version bump.** Edit the `version` in `packages/glass-gl/package.json`,
  commit, and push to `main`. CI (`.github/workflows/publish.yml`) then publishes to npm and
  cuts a GitHub Release — **only when that version isn't already on npm**, so ordinary pushes
  don't republish. This is effectively a lightweight pooled release: pool commits, bump when a
  version is worth cutting.
- `glass-gl` is a **package**, so `npm publish` *is* the release — there is no prod deploy to gate.

**Deferred (until there are users / a real cadence):** full semantic-release automation,
commit-driven auto-versioning, changelog generation, and pooled-release triggers. Reference
skills for that day live in `.claude/skills/` (`conventional-commits`,
`semantic-release-automation`, `pooled-release`, `production-release-gating`).

**`production` branch** exists but is **reserved for the future demo-site deploy** (Vercel
gating — "don't deploy every commit to prod"), NOT for the npm package.

## What this project is

A **glass UI toolkit** distributed through several channels at once, from a single repo:

1. An **AI skill** (`SKILL.md`) teaching how to build glassmorphism / liquid-glass UI.
2. A **CSS package** (`glass.css`) shippable via npm + CDN.
3. A **shadcn registry** so people `npx shadcn add <url>` the React components.
4. A **Claude Code plugin** (installable via a marketplace manifest).
5. A **docs + demo website** (Next.js on Vercel) modeled on glasscn-components.vercel.app.

The unique angle: glass at **three fidelity tiers**, in one place —
- **Tier 1 — CSS `backdrop-filter` blur** (frosted; what most libraries stop at)
- **Tier 2 — SVG `feDisplacementMap`** (real refraction/bending, no canvas)
- **Tier 3 — WebGL fragment shader** (per-frame refraction lens; showpieces/exports)

Most competing libraries are tier-1 only. Tiers 2–3 are the differentiator — protect them.

## Naming status (DECISION PENDING — do not hardcode prematurely)

The product name is **not finalized**. Shortlist: `Refract` / `Vitre` / `Lucent Glass` /
scoped `@wiiiimm/glass-ui`. Until decided, the repo folder is `glass-ui-kit` (placeholder).
When the name lands, it must be set consistently in: `package.json`, the plugin manifest,
the marketplace manifest, the shadcn registry, the site, and the skill's `name:` field.
**If you touch publishing config, check whether the name is still a placeholder first.**

## Repository layout

Current (built):
- `plugins/glass-ui/skills/glass-ui/` — the canonical **skill** (SKILL.md, README, references/, assets/). **Single source of truth** for the glass technique + `glass.css`.
- `plugins/glass-ui/.claude-plugin/` and `.claude-plugin/` — Claude plugin + marketplace manifests (to be written).
- `src/ui/` — canonical React component source (to be written).
- `public/` — built static output for Vercel (registry JSON, demo) (to be generated).

Planned (not yet built):
- `site/` (or `app/`) — Next.js docs + demo website (Fumadocs or Nextra + shadcn).
- `registry.json` + `public/r/*.json` — shadcn registry, built with `shadcn build`.
- `.github/workflows/` — CI to publish npm, deploy the site, rebuild the registry.

## Canonical sources & the no-drift rule

- The **CSS** lives once at `plugins/glass-ui/skills/glass-ui/assets/glass.css`. Other
  channels (npm, site, registry) must **copy from** it via a build step, never fork it.
- The **skill** is the source of truth for technique docs. The website's docs should be
  generated from / kept in sync with the skill's `references/*.md`, not rewritten.
- If you change the glass recipe, update `glass.css` first, then propagate.

## Conventions

- Always ship the `-webkit-` prefix for `backdrop-filter`, and the `@supports` /
  `prefers-reduced-transparency` fallbacks. Glass without fallbacks is a bug here.
- Tier 2 (SVG `url()` on backdrop-filter) is **flaky in Safari** — always provide a blur
  fallback. Never ship a refraction demo that's blank in Safari.
- Glass is **only visible over a busy/colorful background** — every demo/example must place
  glass over a gradient or image, never a flat color.
- Keep `SKILL.md` lean; put depth in `references/`. Match existing file style.

## How to verify changes (glass is visual — look, don't just read)

```bash
# serve the skill's live demo
python3 -m http.server 8000 --directory plugins/glass-ui/skills/glass-ui/assets
# open http://localhost:8000/example.html — check: backdrop visibly blurs (tier 1),
# the refraction card bends the gradient (tier 2), text stays legible, rim highlight shows.
```
Test in both a Chromium browser and Safari (tier-2 fallback) before claiming it works.

## Distribution mechanics (for reference)

- **npm/CDN**: `npm publish` the CSS; usable via jsDelivr/unpkg with no install.
- **shadcn**: `shadcn build` compiles `registry.json` → `public/r/*.json`; users run
  `npx shadcn add <site-url>/r/<item>.json`. "Publishing" = deploying the site.
- **Claude plugin**: the repo *is* the marketplace (via `.claude-plugin/marketplace.json`);
  users add it with `/plugin marketplace add <repo>`. No separate publish step.
- **Skill marketplaces** (mcpmarket, lobehub, claudemarketplaces): these **index public
  repos**; you don't push to them. Getting listed is a one-time submission / they auto-crawl.

## Reference material (not part of the shipped repo)

The competitive research and downloaded peer projects live OUTSIDE this repo at
`../glass-ui/_reference/` with a comparison in `_reference/NOTES.md`. The original studied
liquid-glass card is at `../glass-ui/bubbbly/`. Do not copy peer code into this repo; study
and reimplement.
