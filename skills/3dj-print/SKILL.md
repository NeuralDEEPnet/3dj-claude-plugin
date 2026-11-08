---
name: 3dj-print
description: Get a model ready to print and, where available, request an operator print quote from 3DJ (Neural Deep Network Ltd). Checks watertightness, wall thickness, overhangs and bed fit against the 3DJ gates, finds the best print orientation, suggests slicer starting points, and manages quote requests and acceptance honestly — no payment, booking or shipping happens here. Use for "is this printable?", "check this STL", "how should I print this?", "will it fit my Prusa / Bambu bed?", "get me a quote to print it", "what would it cost to have it printed", or after 3dj-make or 3dj-design when the person wants a physical part.
---

# 3DJ Print — printable, then printed

Two separate jobs. Part A is free and always available. Part B needs a signed-in 3DJ account and only creates review requests and records acceptances.

Every check runs on 3DJ's own tools; nothing runs on the person's machine. Rules and fixes: `../3dj-make/references/dfam.md`. Tools: `../3dj-make/references/tools.md`.

## A · Print-readiness

1. **Get the geometry into 3DJ**:
   - A B-Rep script (ours or the person's): check it directly.
   - A saved 3DJ model: `cad_list_models` / `list_my_models` gives the `modelId` and script.
   - An STL, STEP or GLB file, by public URL or as a file under 20 MB (account): `cad_import_model` stores it privately and returns a `modelId`, its bodies and bounding box. Check it through a one-line script that returns `importedModel("<modelId>")`.
   - OpenSCAD source (account): `openscad_compile_model` with `buildVolumeMm` for the closed-mesh check and printer fit; numbers only.
2. **Get the printer**: bed size, nozzle, material. Assume FDM, 0.4 mm, PLA or PETG, 220 × 220 × 250 mm when unknown, and say so. Common beds: Prusa MINI 180 × 180 × 180, Prusa MK4 250 × 210 × 220, Bambu A1 mini 180³, Bambu A1 / X1 / P1 256³.
3. **Check**: `cad_validate_script` for size, mass, closed bodies and bed fit, then `cad_render_views` with `highlight: "thin-walls"` and again with `highlight: "overhangs"`. For a multi-part model, check each part on its own.
4. **Orient**: render the overhang highlight for two or three candidate poses (rotate the part in the script) and pick the one with the least support, a flat face on the bed and a sensible height. Then apply judgement the renders cannot: layer lines along the main load path, cosmetic face on the bed or facing up, small footprints that need a brim. For a B-Rep part, `cad_export_model` with `format: "stl-fins"` and `supportFins` can bake the pose and breakaway supports into the STL (account).
5. **Fix list**: every problem paired with a concrete change — "wall near [12, 40, 18] is 0.9 mm → set `wall = 1.6`", "flat ceiling at z 22 → 45° chamfer", "does not fit 180 mm bed → split along the back plane with two 4 mm pins". When the source is ours, make the change and re-run the check.
6. **Deliver**: the check images, the chosen orientation, the fix list, the exported STL when one was made, and slicer starting points from `dfam.md` — stated as starting points. Solid mass from `measuredGeometry` is an upper bound; a sliced part is lighter.

## B · 3DJ operator quote (review only)

Three gates stay separate, and none is implied by another:

- **Design** — the person approved the model revision and its functional dimensions.
- **Manufacturing** — a real exported file inspected for watertightness, walls, overhangs, bed fit and assembly, plus a printer- and material-specific operator review. `cad_validate_script`, `cad_render_views` and `cad_get_evidence_schema` support this gate; they do not pass it.
- **Commercial** — an operator-issued, expiring EUR quote tied to the inspected file hash, quantity, material, colour, destination, taxes and shipping. Never estimate a price or lead time as if it were a quote.

Steps:

1. `get_account_status` → check sign-in and `printQuoteRequestsOpen`.
2. The quote needs a **completed saved model in the person's 3DJ gallery**. A B-Rep script can be saved directly with `cad_save_model` (it returns the `modelId`). A local `.scad` or `.stl` is not a saved model: the person opens `https://www.3dj.app/workspace`, pastes the OpenSCAD source (or imports the file), and saves it. Then find it with `list_my_models` / `cad_list_models`.
3. When requests are open and the person asks to submit, collect `modelId`, `dimensionsMm`, `material`, `colour`, `quantity` (1–100), `destinationCountry` (two letters), `destinationRegion` (city or postal region only — never a street address), `intendedUse`, optional `notes`, and call `request_print_quote` with a new `requestKey` (e.g. `print-phone-stand-001`; reuse it only for an identical retry). Report the request id, `pending_review` status, and that no payment is due now.
4. When requests are closed, say so and give a manufacturing brief the person can keep or email to 3d@neuraldeep.net: model, revision, dimensions, material, colour, quantity, use, region, and the check findings.
5. `list_print_requests` for status and operator notes. `cancel_print_request` only when the person asks.
6. `list_print_quotes` → show material, colour, quantity, dimensions, region, unit price, shipping, tax, EUR total, expiry, lead time, and the operator's inspection note with the file SHA-256 — the inspection note is the operator's attestation.
7. Only after the person explicitly accepts those exact terms, call `accept_print_quote` with `quoteId` and `quoteHash`. Report the recorded acceptance and that payment checkout, print booking and shipping are not connected in this release.

Never collect card details, full addresses, or credentials in chat or tool arguments.
