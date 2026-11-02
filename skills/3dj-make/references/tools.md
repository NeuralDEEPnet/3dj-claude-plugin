# 3DJ tools, routes and limits

Verified against the live 3DJ MCP server on 1 November 2026 (23 tools, 6 of them public; endpoint `https://www.3dj.app/api/mcp`, with sign-in on 3dj.app). The live tool list is authoritative: if it differs from this page, follow the live schemas and never invent a tool.

## How the tools appear

The same tools can reach this session three ways. Use whichever is present; the arguments are identical.

| Route | Tool names look like | Notes |
|---|---|---|
| This plugin's `3dj` server (Claude Code) | `mcp__plugin_3dj-studio_3dj__openscad_compile_model` | Sign in once: run `/mcp`, choose **3dj**, **Authenticate**. |
| Claude app connector "3dj" | `mcp__3dj__openscad_compile_model` | Sign in under Settings → Connectors → 3dj. |
| Adaptive gateway | `threedj_cad_and_print` with `operation`, or `adaptive_proxy_mcp_tool` with `app: "3dj"`, `action: "call_tool"`, `toolName`, `arguments` | The shortcut covers starters, validation, compile, saved CAD models, account and quotes; use the proxy for generation and the rest. |

Tool calls may be deferred: load them through the session's tool search before the first call.

## Free tools — no account

| Tool | Arguments | Returns |
|---|---|---|
| `openscad_get_starters` | — | 3 OpenSCAD starters (`mounting-plate`, `vented-cup`, `box-lid`) and the language rules 3DJ's engine accepts. |
| `cad_get_starters` | `index?` (true → ids, titles and summaries only), `category?` (e.g. `hinge`, `enclosure`, `gear`, `fastener`, `mating`, `bracket`, `joint`; a word that matches nothing returns all) | 19 Replicad/OpenCascade B-Rep starter scripts with `Param.*` declarations, plus the full DSL reference (`dslReference`). Call with `index: true` first, then fetch the one starter you need by a `category` word: the full catalogue is large. |
| `cad_validate_script` | `code` | Builds a B-Rep script in 3DJ's isolated kernel and measures it: syntax and `Param.*` definitions, then `buildValidation`, `measuredGeometry` (bodies, size, volume, bed fit, checks) and per-body `parts`. Never creates a file. |
| `cad_get_evidence_schema` | — | The Design Evidence Pack v1 gates: watertight, 0.8 mm nozzle-safe wall, 1.2 mm recommended wall, 45° overhang, 220 × 220 × 250 mm bed, PLA 1.24 g/cm³, evidence labels. |
| `compose_design_brief` | `code`, `parameters?`, `productName?`, `intendedUse?`, `material?`, `finish?`, `colour?`, `style?` | Builds and measures a B-Rep script and returns import steps for Blender, FreeCAD, Fusion 360, Onshape and slicers, plus a render brief. Renders, sends and saves nothing. |
| `cad_get_configurator` | `code` or `modelId`, `parameters?` | Builds the script and returns what a live three.js configurator needs: the `Param.*` schema (type, default, min, max, step, unit, choices), parts with colours and materials, connectors and `placeAt` joints, BOM, measured geometry and pinned CDN import URLs (`cdnImports`). Read-only. In Claude hosts that support interactive tool results it also shows the live configurator inline (`ui://3dj/configurator`); `cad_get_starters`, `cad_validate_script`, `cad_save_model` and `cad_publish_configurator` link the same view. |

The two tools that build in 3DJ's kernel, `cad_validate_script` and `compose_design_brief`, are throttled per caller address when called without an account: 20 builds a minute and 120 an hour, answered with `rate_limited` and `retryAfterSeconds`. Signed in, they use the account's own allowance of 90 builds an hour and 500 a day.

## Account tools — OAuth sign-in (or a per-user API key in hosts that use headers)

| Tool | Arguments | Notes |
|---|---|---|
| `openscad_compile_model` | `code`, `parameters?` {name: number}, `buildVolumeMm?` [x,y,z] | Real OpenSCAD (Manifold) in an isolated worker: effective source, clamped parameter values, triangle count, closed-mesh check, bounding box, printer fit. 20/hour, 100/day. Creates **no** STL and no gallery record. |
| `cad_list_models` | `limit?` 1–100 | The user's saved parametric and photo-CAD models with script source and parameter values. |
| `cad_export_model` | `code`, `parameters?`, `format?` (`stl` default, `step`, `glb`, `zip`, `stl-fins`, `stl-fins-only`), `partName?`, `name?`, `supportFins?` | Builds a B-Rep script in 3DJ's server kernel, measures every body and writes a real file: binary STL, ISO 10303 STEP, GLB (one named, coloured node per part) a ZIP of per-part STLs with a BOM, or an STL with breakaway Support Fins and a bed pad baked in (`stl-fins`; `supportFins` sets pose, nozzle, layer height, gap, pad and cutouts). Returns `downloadUrl`, `sha256`, sizes, volumes and import steps. 30/hour, 150/day. **The download link is reachable by anyone who has it.** No gallery record. |
| `cad_save_model` | `code`, `name`, `parameters?`, `summary?` | Builds the script, checks it is closed, and saves it to the user's gallery as an editable model with an STL. Returns `modelId`, `downloadUrl` and `openUrl` (reopens it in the workbench with these parameters). 20/hour, 100/day. Needed before a print quote request. |
| `cad_publish_configurator` | `code`, `name`, `parameters?`, `summary?`, `ui?` (`title`, `theme`, `showExplode`, `showBom`) | Builds and saves the part to the user's gallery and returns a shareable live configurator `shareUrl` (`https://www.3dj.app/configurator/<id>`) and `openUrl`. **Anyone with the share link can open it.** Only when the person asks to publish. |
| `list_providers` | `force?` | Health of each generation backend, supported inputs, remaining hourly/daily generation allowance. |
| `generate_from_text` | `prompt`, `negativePrompt?` | Starts an asynchronous concept mesh (used by the 3dj-concept skill; 3DJ first creates one AI reference image from the prompt, then reconstructs the mesh). Returns `modelId`, `status: "processing"`. Uses allowance. |
| `generate_from_image_url` | `imageUrl` (public HTTPS, ≤ 12 MB), `name?` | Image-to-3D. Same polling. |
| `get_generation_status` | `modelId` | `processing` / `complete` / `failed`, progress, provider, `downloadUrl` when complete, or the real failure reason. |
| `list_my_models` | `limit?`, `status?` (`complete`/`failed`/`processing`) | The user's models, newest first. |
| `get_model_download_url` | `modelId` | Absolute `downloadUrl`, actual `format` (`glb` or `stl`), `refinementHandoff`. |
| `get_account_status` | — | Account id, auth method, scopes, saved-model counts, `printQuoteRequestsOpen`. |
| `request_print_quote` | `requestKey`, `modelId`, `dimensionsMm` [x,y,z], `material`, `colour`, `quantity` 1–100, `destinationCountry` (2 letters), `destinationRegion` (city or postal region only), `intendedUse`, `notes?` | Creates a review request only (`paymentRequiredNow: false`). Needs a completed saved model. Reuse `requestKey` for identical retries; new key for any change. |
| `list_print_requests` | — | Requests and operator notes. |
| `cancel_print_request` | `requestId` | Only when the user asks. Not a refund or order cancellation. |
| `list_print_quotes` | — | Operator-issued EUR quotes: item, shipping, tax, total, expiry, lead time, inspection note, file SHA-256, `quoteHash`. |
| `accept_print_quote` | `quoteId`, `quoteHash` | Records acceptance of the exact displayed terms. No charge, booking or shipment. |

## Sign-in and safety

- Never ask for an API key or token in chat and never put credentials, `userId` or tokens in tool arguments — the server rejects them (`credentials_in_arguments`).
- The server also refuses sensitive personal data (government IDs, medical records, biometrics, card numbers) in any argument. None is needed for CAD.
- When a call returns 401 or "connect your account", say which route above signs in, and carry on with the free path meanwhile.

## Workbench links

| Purpose | Link |
|---|---|
| Open the workbench (OpenSCAD and B-Rep editors, STEP/STL export, gallery save) | `https://www.3dj.app/workspace` |
| Reopen a saved model | `https://www.3dj.app/workspace?model=<modelId>` (`cad_save_model` returns it as `openUrl`) |
| A published live configurator | `https://www.3dj.app/configurator/<modelId>` (`cad_publish_configurator` returns it as `shareUrl`) |
| Open a generated mesh in the viewer | `https://www.3dj.app/workspace?modelUrl=<URL-encoded downloadUrl>` |
| Start image-to-3D from a reference image (e.g. a product render) | `https://www.3dj.app/workspace?ref=<URL-encoded image URL>` |
| Mint a per-user API key (CLI and header-based hosts) | `https://www.3dj.app/agents` |

Unsaved source has no deep link: for an OpenSCAD script, or a B-Rep script the person doesn't want saved, tell them to open the workbench, pick the matching editor and paste the file.

## The free local bench

`scripts/bench.mjs` (in the 3dj-make skill folder) needs only Node 18+. It compiles OpenSCAD with the official WebAssembly build (OpenSCAD 2025.01, Manifold backend — the same backend as `openscad_compile_model`), which `node BENCH setup` downloads once into `~/.cache/3dj-bench` (override with `THREEDJ_BENCH_HOME`) after the person agrees. It reads STL and GLB too. Use it for every iteration; use `openscad_compile_model` when an official 3DJ measurement is wanted or when Node is unavailable.
