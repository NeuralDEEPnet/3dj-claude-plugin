---
name: 3dj-design
description: Design a physical object end to end with 3DJ as the live CAD kernel — brief, design DNA, three genuinely different form directions modelled and compared side by side at the same scale, critique, refinement and a printable handoff. Use for "design a lamp / headphone stand / planter / speaker / desk organiser / robot / controller…", "give me some directions for…", "make it look premium / bold / playful", "help me design something cool I can 3D print", or whenever someone wants real options to choose between. Part of the Neuraldeep Design Studio; builds and renders every model in 3DJ.
---

# 3DJ Design — concepts you can see, compare and print

Work like a studio: decide what the object must do and be, explore genuinely different answers as real models, choose with evidence, then make it real. If the Neuraldeep Design Studio skills are installed (`3d-object-design`, `design-core`, `design-director`), follow their process, templates and depth modes and use this skill for the modelling, comparison and print stages. Otherwise follow the compact flow below.

Every model is built, measured and rendered by 3DJ's own tools; nothing runs on the person's machine. Modelling rules: `../3dj-make/references/brep-cad.md`; tools and limits: `../3dj-make/references/tools.md`; print rules: `../3dj-make/references/dfam.md`.

## Choose the depth — say it in one line

| Depth | When | Directions |
|---|---|---|
| Rapid | clear, small ask; "just make me…" | 1 direction + 2–3 proportion variants |
| Standard (default) | most design requests | 3 directions |
| Full | a new product, a family, something expensive to get wrong | 4–5 directions + a validation plan |

## 1 · Brief and package

- Who uses it, where, doing what; the one to three things it must do well; constraints (size, printer, materials, budget, parts it holds).
- **Package**: every real component with its size (phone 78 × 165 × 9 mm in case, USB-C cable 6 mm, 2 × M3 inserts …), tagged measured / observed / inferred / assumed. Assumed sizes open the "confirm before printing" list.
- Ask at most three questions, and only when a wrong guess would be expensive. Otherwise state assumptions and move.
- Never invent user research. Insights not from the person are hypotheses — label them.

## 2 · Design DNA — the filter

Write 3–5 principles and 2–3 anti-principles, each concrete enough to reject an attractive design ("one continuous sweep from base to cradle"; anti: "no generic rounded box", "no visible screws on the front"). Translate vague words into form language: "premium" → tight radii family, a single crisp shadow gap, mass low, matte surface; "playful" → one exaggerated proportion, a tilt, a bright accent part. State the translation.

## 3 · Three directions that differ in logic

Each direction is a different structural idea — how it stands, where the mass sits, how the parts split, how it holds the thing — not three stylings of one idea. Name each with one word that carries its idea (for a lamp, say: Crane, Lantern, Monolith).

1. For each, write a short parametric B-Rep blockout (`a-<name>.js`, `b-<name>.js`, `c-<name>.js`, about 20–60 lines): primary volumes and the critical interface only (the slot, the bore, the mounting face), plus a simple **ghost** of the held object as its own named part behind a `Show ghost` boolean parameter (headphones, phone, plant pot) so fit and scale are visible. Start from `cad_get_starters`. Same units, all on Z = 0.
2. Render them the same way: for each direction, `cad_render_views` with the ghost on, the same `views` (`iso` and `front`; add `right` when the ideas live in the side profile, such as cantilevers and leans), the same `size` and `style: "shaded-edges"`. Then `cad_validate_script` for each to get size, mass, bodies and the wall, overhang and bed checks.
3. Look at the views side by side: the front views show stance and proportion, the iso views show mass, character and fit around the ghost. Compare sizes from `measuredGeometry`, not from the pictures, since each render is framed to its own part. If a direction is weak in silhouette, fix the idea now — detail will not rescue it. A bed fail at this stage is information (split it, or rethink proportions), not a reason to drop a direction.
4. Show the sheet with one line per direction: the idea, its strongest point, its biggest risk.

## 4 · Critique, then choose

- When the `3dj-critic` agent is available, give it the rendered views, the `measuredGeometry` of each direction, the brief and the DNA for an independent review. Otherwise review through five lenses yourself — function, form against DNA, ergonomics and scale, printability, cost and time (the measured mass and part count are the evidence) — and make every lens say at least one uncomfortable thing.
- Recommend one direction or a synthesis, traced to the brief and DNA. If the person is present, let them choose; if not, continue with the recommendation and say so.

## 5 · Develop the chosen direction

Large to small: stance → silhouette → proportion → section → transitions → details. After each meaningful step, build with `cad_validate_script` (`preview: true`) and look at the picture, then `cad_render_views` with `highlight: "thin-walls"` and `"overhangs"` as the print check, and `cad_check_assembly` for multi-part products. Review the product as it will look with the ghost on; export with it off.

Show each change before and after with `cad_compare_revisions`. Settle key proportions by comparing variants instead of guessing: `cad_compare_revisions` with the same code at two values of the parameter (for example `Height` 140 against 170). Add the real interfaces, clearances (0.25 mm), and the print strategy: an orientation per part chosen from the overhang views (`../3dj-make/references/dfam.md`), and a split into parts where a single print would need heavy support or exceeds the bed.

## 6 · Finish: CMF, print, handoff

- **CMF**: name the material and finish (matte PLA, PETG, ASA outdoors, TPU feet or grips), colour as a decision (two-tone by part or by a filament change at a set height), layer height for visible curves, surface treatments (fuzzy skin for grip).
- **Optional next steps** — offer once: an operator print quote with the 3dj-print skill; an organic or sculptural variant with the 3dj-concept skill; a live configurator of the chosen direction with the 3dj-configurator skill.
- **Handoff**: for a B-Rep product, a dimensioned drawing from `cad_export_drawing` when someone else will make it; review notes can be pinned to the saved model with `cad_add_review_comment` when the person works with a team. Then the views and check images, the `.js` source, the exported files per part (`cad_export_model`, account), the knob table, key dimensions with evidence tags, assumptions to confirm, print settings, and the next prototype (e.g. "print only the cradle section at 30 mm tall to test the phone fit").

## Rules

- Directions differ in core logic, not colour.
- One DNA across every part of the object.
- 3DJ's checks screen geometry against the 3DJ gates; it does not certify strength, safety or manufacturability.
- Keep the conversation light: pictures first, short captions, decisions stated plainly.
