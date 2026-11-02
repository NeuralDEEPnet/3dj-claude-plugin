# Changelog

## 0.4.0 — 2026-11-02

- New `3dj-configurator` skill: turns a parametric model into a live three.js configurator whose sliders rebuild the real CAD solid. It writes configurator-friendly parameters (customer-facing knobs, safe derived values, joints as knobs) and validates the extremes. It shows the configurator inline in Claude (3DJ's `ui://3dj/configurator` view), publishes a share link with the person's OK (public to link holders), or hands over an embeddable `<threedj-configurator>` page, using the npm runtime for Claude artifacts once published and www.3dj.app's runtime elsewhere.
- Tool reference notes the inline configurator view; `3dj-design` offers a configurator of the chosen direction; README and disclosure updated; configurator trigger evals.

## 0.3.2 — 2026-11-01

- The `3dj` MCP server now points at 3DJ's own address, `https://www.3dj.app/api/mcp`, with OAuth sign-in on 3dj.app (all five scopes requested at sign-in; HTTP 403 step-up for a missing scope). The old adaptive.ai address stays live as an alias, but people who installed an earlier version should sign in to 3dj again after updating.

## 0.3.1 — 2026-11-01

- Tool reference matches 3DJ's 23-tool server (6 public): `cad_get_starters` `index` mode and working category words, `cad_get_configurator` and `cad_publish_configurator` for live three.js configurators, Support Fins STL exports (`stl-fins`, `supportFins`), and the configurator share link. The endpoint stays on the adaptive.ai address until 3DJ's 3dj.app address is confirmed live.

## 0.3.0 — 2026-11-01

- New `3dj-concept` skill, folded in from the separate 3DJ Concept plugin: organic and sculptural concept meshes from a text prompt or a public image URL, followed to completion, checked on the bench and handed over with a workbench link. It tells people up front that text-to-3D creates an AI reference image on 3DJ's servers.
- README and disclosure updated for generation; `3dj-make` and `3dj-design` point to the skill directly; concept trigger evals added.

## 0.2.2 — 2026-11-01

- Matches 3DJ's server update of 1 November: `cad_validate_script` always builds and measures the solid, so the skills no longer hedge on it, and the free building tools' limits are documented (per-address without an account, 90 builds an hour and 500 a day with one).
- B-Rep reference covers the new language: `placeAt` joints driven by a parameter, involute gears and belt drives, sweeps along 3D curves and constraint-solved sketches, with a second live-validated example (a parametric gear pair). Notes that `partName` must match the part names the build reports.
- `cad_get_starters` filtering explained (matched against id, title and summary; 19 starters, including the new hinged-lid enclosure).

## 0.2.1 — 2026-11-01

- Claude Code asks for all five 3DJ scopes at the first sign-in (`oauth.scopes` in `.mcp.json`), so later tools don't stall on a missing scope.
- Directory listing fields in `plugin.json`: icon, documentation, support, privacy and terms URLs.

## 0.2.0 — 2026-10-31

- Uses 3DJ's new server tools: `cad_export_model` returns real STL, STEP, GLB or per-part ZIP files for B-Rep parts, and `cad_save_model` saves a part to the gallery with a workbench reopen link (also the path to a print quote). Skills choose the export format by destination and tell people that export links are public to anyone who has them.
- Tool reference updated to the live 21-tool catalogue (5 public), including `compose_design_brief` and the `/workspace?model=<id>` reopen link.

## 0.1.0 — 2026-10-31

First release of 3DJ Studio, the CAD bench of the Neuraldeep Design Studio.

- Connects the 3DJ MCP server: OpenSCAD and B-Rep starters, script validation and the evidence schema without an account; hosted OpenSCAD compile, saved models, downloads and print quote reviews with one.
- Skills: `3dj-make` (functional printable parts), `3dj-design` (three modelled directions compared at one scale, critique, development, handoff), `3dj-print` (print-readiness, orientation, slicer starting points, operator quote reviews).
- Agent: `3dj-critic`, an independent review of rendered models against the brief and design DNA.
- Free local bench (`skills/3dj-make/scripts/bench.mjs`, MIT, Node 18+): compiles OpenSCAD 2025.01 with the WebAssembly engine (downloaded once by `setup`, pinned and SHA-512 verified, install scripts disabled), reads STL and GLB, checks watertightness, wall thickness, overhang regions, bed fit and mass against the 3DJ Design Evidence Pack v1 gates, and renders views, print checks, orientation studies and same-scale concept or parameter-variant comparisons.
- Form cookbook of 13 tested OpenSCAD patterns; package-ghost and assembled-review conventions.
- Evals: two graded cases and trigger queries.
