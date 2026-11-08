---
name: 3dj-make
description: Make a functional, 3D-printable part with 3DJ — brackets, mounts, enclosures, cases, holders, stands, adapters, knobs, organisers, jigs and replacement parts. Writes a parametric model, builds and measures it in 3DJ's CAD kernel, looks at real rendered views and print-check highlights, fixes problems, and hands over the files with named knobs to adjust. Use for "make me a bracket for…", "design a case for this PCB", "I need a replacement knob, 22 mm shaft", "model a phone stand I can print", "turn these measurements into a part", or any printable object with real dimensions. For a product designed through compared concepts use 3dj-design.
---

# 3DJ Make — a printable part, measured and seen

Go from request to a parametric part the person can see, adjust by name and print — in one pass, with the picture shown early and questions kept to the minimum.

## 0 · Know the tools (check silently)

Every build, measurement and picture comes from 3DJ's own server. Nothing is installed or run on the person's machine.

- **Free, no sign-in** (throttled per conversation): `cad_get_starters`, `cad_validate_script` (with `preview: true` for a picture), `cad_render_views`, `cad_check_assembly`, `cad_compare_revisions`, `cad_get_evidence_schema`, `openscad_get_starters`.
- **With a 3DJ account**: `openscad_compile_model`, `cad_export_model` (real STL, STEP, GLB files), `cad_save_model`, `cad_export_drawing`, `cad_import_model` and the rest.
- Names, routes (plugin server, Claude app connector, Adaptive gateway), limits and sign-in: `references/tools.md`.
- **No 3DJ tools reachable at all**: write the source carefully, say plainly that it has not been built or checked, and give the person the 3DJ workbench step (`https://www.3dj.app/workspace`).

## 1 · Pin the part down (fast)

1. Write a three-line brief: what it does, what it touches (the phone, the 25 mm pipe, the M3 holes 40 mm apart), where it lives.
2. List the dimensions that make or break the fit and tag each **measured / observed / inferred / assumed**.
3. Ask at most two questions, and only for fit-critical numbers you cannot sensibly assume ("What is the pipe's outside diameter?"). Otherwise state the assumption and continue.
4. Note printer bed and material if known (default: FDM, 0.4 mm nozzle, PLA or PETG, 220 × 220 × 250 mm bed).

## 2 · Pick the engine

| The person needs | Use |
|---|---|
| A printable part with pictures at every step (default), STEP for other CAD or CNC, true edge fillets, mating assemblies with standard hardware, gears, a hinge or another joint, or a part saved to the 3DJ gallery | 3DJ B-Rep — follow `references/brep-cad.md` |
| OpenSCAD source specifically (they asked for it, or will edit it in OpenSCAD) | OpenSCAD — follow `references/openscad.md`; `openscad_compile_model` (account) compiles and measures it, but returns no pictures and no file |
| Organic, sculptural, character, "make this photo 3D" | the 3dj-concept skill for a generated mesh; or sculpt in B-Rep with lofts, sweeps and revolves |

## 3 · Model it

1. Start from something proven: a 3DJ starter (`cad_get_starters` with `index: true`, then a `category` word); for OpenSCAD, `openscad_get_starters` or a pattern in `references/form-cookbook.md`. Do not start from a blank page.
2. Write `<name>.js` (B-Rep) or `<name>.scad` (OpenSCAD) into a project folder (`3dj/<name>/` unless the person's project has a place for it). Top-level parameters with honest ranges and plain names, comments marking measured versus assumed sizes, walls ≥ 1.2 mm, 0.25 mm clearances, separate named parts for multi-part prints, nothing below Z = 0.
3. When the part holds or fits something, model a simple **ghost** of it (the phone, the pipe, the PCB) as its own named part behind a `Show ghost` boolean parameter. It is the fastest way to prove fit and show scale. Leave it off for exports.
4. Give it character on purpose: choose an edge strategy (soft radii, crisp chamfers, or one family of each) and one dominant gesture. A bracket can still look designed.

## 4 · Build, measure, look — every iteration

1. `cad_validate_script { code, parameters, preview: true }`. Read `buildValidation` and `measuredGeometry` (bodies, size, volume, mass, closed, bed fit, checks). Fix any failure before showing anything.
2. **Look** at the preview image properly: does it match the brief, are the proportions right, do holes go through, are parts separate, is anything floating or fused that should not be? Fix and rebuild. The person should see a good version first, not the first version.
3. **Print check**: `cad_render_views { code, parameters, views: ["iso", "front", "right"], highlight: "thin-walls" }`, then again with `highlight: "overhangs"`. Use `style: "section"` with a `sectionPlane` to see inside walls and bores. Fix thin walls and large unsupported overhangs (45° chamfer instead of a bottom fillet, teardrop holes, a flat underside) — see `references/dfam.md`.
4. Parts that meet or move: `cad_check_assembly`. Fix any interference before showing the assembly.
5. To show what a change did, or to compare two values of one parameter: `cad_compare_revisions` with `before` and `after` returns the deltas and an overlay image.
6. **OpenSCAD source**: with an account, `openscad_compile_model { code, parameters, buildVolumeMm }` for the compile, closed-mesh check and printer fit. It returns numbers only, so say that no picture exists yet and offer the workbench to see it.

These builds are throttled without sign-in (`references/tools.md`): on `rate_limited`, wait the given time and don't loop. Suggest signing in for long sessions.

## 5 · Show it and hand it over

Lead with the rendered image — it is the answer. Then, briefly:

- **Files**: the source file (the source of truth) and the images. With an account, `cad_export_model` writes the real STL, STEP or GLB (say that its links work for anyone holding them). Without one, the person pastes the script into `https://www.3dj.app/workspace` and exports there. Never say a file exists until a download URL came back or the person has exported it.
- **Knobs** — a small table the person can act on by name:

  | Knob | Now | Range | What it changes |
  |---|---|---|---|
  | `Slot width` | 12 mm | 8–20 | fits phone + case |

- **Print notes**: orientation (from the overhang view and `references/dfam.md`), supports, material, approximate mass from `measuredGeometry` (solid mass is an upper bound; a sliced part is lighter).
- **To confirm**: every **assumed** dimension, and a quick fit-test print when one matters.
- **Next moves**: two or three one-liners the person can just say back ("make it 10 mm taller", "add a cable notch", "show me three directions").
- **Drawing** (on request, account): `cad_export_drawing` makes a dimensioned A4 sheet (front, top, right, isometric, ISO 7200 title block) as PDF, SVG or DXF for a machinist or a supplier.
- **Keep it in 3DJ** (mention once): `cad_save_model` saves it to the gallery with a reopen link, and is needed before a print quote with the 3dj-print skill.

## 6 · Iterate by name

Map plain requests to parameters ("wider" → `Width`), change the default in the file so the source stays the truth, rebuild, and show the new picture with what changed (`cad_compare_revisions` when the change is subtle). Add a parameter when the person keeps asking for the same kind of change.

## Honesty

- 3DJ's checks are geometry screens against the 3DJ gates, not manufacturing, structural, food-contact or safety certification. Say which checks ran; never just "print-ready".
- Never claim a STEP or STL exists until it has been written, or that 3DJ built something unless the tool returned a result.
- Never ask for API keys or tokens in chat, and never put credentials in tool arguments.
