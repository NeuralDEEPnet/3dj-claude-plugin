# Maintaining 3DJ Studio

This repository is the plugin: the repository root is the plugin folder that the Claude directory installs, so everything committed here ships to users. Keep it to the plugin's own files.

## Layout

| Path | What it is |
|---|---|
| `.claude-plugin/plugin.json` | Manifest. Raise `version` with every release |
| `.claude-plugin/marketplace.json` | Lets people install straight from this repository |
| `.mcp.json` | The 3DJ MCP server. Must point at the same URL as 3DJ's connector listing |
| `skills/3dj-make/` | Functional parts, plus the shared references and the bench (`scripts/bench.mjs`) |
| `skills/3dj-design/`, `skills/3dj-print/`, `skills/3dj-concept/` | Studio flow, print-readiness and generated concept meshes; they use the bench and references in `3dj-make` |
| `agents/3dj-critic.md` | Independent review agent |
| `evals/` | Graded cases and trigger queries for `claude plugin eval` |

## Before every release

```bash
claude plugin validate --profile directory --strict .
```

Then smoke-test the bench (it needs Node 18+):

```bash
node skills/3dj-make/scripts/bench.mjs setup        # once per machine
node skills/3dj-make/scripts/bench.mjs build <part.scad> --bed 220,220,250
node skills/3dj-make/scripts/bench.mjs compare <a.scad> <b.scad>
node skills/3dj-make/scripts/bench.mjs orient <part.stl>
```

Every pattern in `skills/3dj-make/references/form-cookbook.md` must compile watertight and pass the wall, overhang and bed gates; extract the `openscad` blocks and build each one after editing the cookbook.

## Rules that keep the directory listing healthy

- **Disclose behaviour.** The README's "What this plugin runs, sends and fetches" section must stay complete and true. Any new network destination, download or data sent to 3DJ goes there first.
- **The engine download stays explicit.** `bench.mjs setup` is the only code path that downloads anything. Changing the engine version means updating `ENGINE_VERSION` and `ENGINE_INTEGRITY` together (`npm view openscad-wasm-prebuilt@<version> dist.integrity`).
- **No credentials in files**, no reading tokens from the user's environment, no package-manager config files (`.npmrc` and similar).
- **Files** stay text, PNG/JPEG/GIF/WebP images or SVG, each under 256 KiB (images excepted), 512 files at most. No `.DS_Store`, symlinks, submodules or LFS.
- **Skills describe; they don't steer.** Skills must not tell Claude to call connectors the person didn't ask for, and must not tell it to follow instructions fetched at runtime. Server-provided language notes are reference data.
- **Generation stays disclosed.** The `3dj-concept` skill tells people before the first generation that text-to-3D creates an AI reference image on 3DJ's servers, and the README's disclosure section says the same. Keep both whenever generation changes.
- **Honest claims.** Bench and 3DJ results are geometry screens, never manufacturing, structural or safety certification.

## The 3DJ server

The MCP server lives in the 3DJ application, not here. When its tools change, update `skills/3dj-make/references/tools.md` and any skill steps that name the tools, then validate and release.
