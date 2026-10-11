---
name: anatomy
description: Interactive isometric machine figures, upstream-maintained.
version: 1.0.0
author: Ryan (wheresryan22)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [isometric, svg, webgl, 3d, diagram, explainer, illustration, docs, html]
    category: creative
    related_skills: [concept-diagrams, system-atlas, architecture-diagram]
    upstream:
      repo: wheresryan22/anatomy
      path: .
---

# Anatomy (upstream-maintained)

> **Catalog stub.** This entry is maintained upstream at
> [wheresryan22/anatomy](https://github.com/wheresryan22/anatomy): the
> repository root is itself a self-contained skill (`SKILL.md` beside `kit/`,
> `scripts/`, `references/` and `examples/`). `hermes skills install
> official/creative/anatomy` pulls the current tree live from that repo
> (quarantined and scanned like any hub install). This directory holds only
> the catalog metadata, so the vendored copy can never go stale.

Anatomy explains a technical idea by building it as a real, working object:
a cache key becomes a pin-tumbler lock, a shared GPU context becomes a gantry
plotter visiting sixteen wells. The agent invents the machine, writes a parts
list, builds the geometry with the bundled isometric kit, and ships a framed,
interactive figure (title, hint, live readout, calm eased motion) as a
standalone HTML page, a lone `.svg`, or React/Next.js components. Optional
layers add WebGL shaders registered to the isometric planes (fire, water,
caustics, lamp light) and a beta 3D mode for parts that turn in depth. A
geometry audit (`node build.mjs --audit`) and a line-end checker prove the
drawing instead of eyeballing it.

Eight worked examples ship with it, from a ~300-line dial indicator to a
rocket engine in a test stand and an arcade cabinet with a coin chute.

## Prerequisites

- Node.js 20.10+ on `PATH`. The kit uses only Node built-ins; there is
  nothing to `npm install`. Each example builds with
  `node examples/<name>/build.mjs` (add `--light` for the light theme).
- Chrome or Chromium for the verification scripts (`scripts/capture.mjs`,
  `drive.mjs`, `lines.mjs`, `turn-check.mjs`). They look in the usual install
  paths or `CHROME_PATH`, launch it headless and talk to it over a loopback
  DevTools port. The builds themselves need no browser.
- Installs pull the whole upstream tree (~12 MB, ~97 files; most of it is the
  built example pages the skill tells the agent to open). The fetch is pinned
  to one tree SHA, recorded in the bundle metadata.
- The skill's own code has no telemetry or update check; its scripts only
  talk to the headless browser over loopback. That Chrome is started with a
  fresh profile and without `--disable-background-networking`, so Chrome
  itself makes its usual background connections to Google services while a
  capture runs.
- The skill is deliberate about scope: it sizes every request first. A single
  mechanism ("Figure") takes 30 to 90 minutes of agent work; a dense "Hero"
  takes hours, and it declines to start an "Epic" without asking.

## When to prefer another skill

- `concept-diagrams`: flat, minimal educational SVGs in minutes, no build
  step and no audit loop.
- `system-atlas`: explorable isometric maps of a software architecture
  (structures, flows, open questions), not a physical machine.
- `architecture-diagram` (bundled): a quick dark-themed infra/cloud diagram.

Reach for Anatomy when the figure is the centrepiece of a docs page, article
or landing page and the idea maps onto a mechanism someone would recognise.

Full documentation: https://github.com/wheresryan22/anatomy#readme
