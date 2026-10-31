# B-Rep CAD in 3DJ (STEP-grade parts)

Use 3DJ's parametric B-Rep engine (Replicad on OpenCascade, compiled in the browser workbench) instead of OpenSCAD when the person needs:

- a **STEP** file for other CAD tools, CNC, injection-moulding quotes or engineering hand-off;
- true **fillets and chamfers** on selected edges, shells, lofts and sweeps;
- **assemblies** with connectors, clearance fits and standard hardware (screws, nuts, heat-set pockets, 2020 extrusion profiles) from 3DJ's `lib`.

## Workflow

1. Call `cad_get_starters` (free; optionally `category`) and read `dslReference` — it is the authoritative language reference. Start from the closest starter (project box, extrusion foot, bolt flange, hinge, gripper, planetary reducer, cradle, lathe profile …) instead of a blank page.
2. Write the script: `Param.number/bool/choice` declarations first, Z up, millimetres, solids on Z = 0, then geometry, then `return` a shape, an array, or named parts `[{ name, shape, color }]`. Keep mating parts separate and seat them with connectors (`withConnectors`, `matchTo`) or references (`withReferences`, `attachTo`).
3. Call `cad_validate_script` with the code and fix every reported problem. It always checks syntax, safety and parameters; the live server may also build the solid and return `buildValidation` and `measuredGeometry` (body count, size, volume, bed fit). Report exactly what it returned — a build check is not an exported file and not a print inspection.
4. Get real files. With a 3DJ account, call `cad_export_model` with the script, the parameter values and a format chosen by where the file is going: `stl` for slicers, `step` for FreeCAD, Fusion 360, Onshape or SolidWorks, `glb` for Blender and web viewers, `zip` for an assembly (one STL per part plus a BOM). It returns a `downloadUrl` and `sha256`; tell the person the link works for anyone who has it. To keep the part in their gallery (and before a print quote), call `cad_save_model` and give them its `openUrl`. Without an account, hand over the `.js` script and parameter values to paste into the CAD script editor at `https://www.3dj.app/workspace`. Either way, a file exists only once a download URL has been returned or the person has exported it.
5. Look at the real output: run the exported STL through the bench (`node BENCH build part.stl`) for views and the print check, when it can be downloaded into the workspace.

## Quick reference (see `dslReference` for everything)

```js
// Validated with cad_validate_script (solid build passed: 1 body, 80 × 40 × 6 mm).
const width = Param.number("Width", 80, { min: 50, max: 160, unit: "mm" });
const screw = Param.choice("Screw", "M3", ["M3", "M4"]);
let plate = roundedRect(width, 40, 4).extrude(6);
plate = fillet(plate, 1, { atZ: 6 });                      // round only the top edges
const cutter = lib.screwCutter(screw, { head: "countersunk", length: 8 });
plate = plate.subtract(cutter.translate(-width / 2 + 10, 0, 6), cutter.translate(width / 2 - 10, 0, 6));
return [{ name: "Plate", shape: plate, color: "#5b6b7a" }];
```

- Primitives: `box(w,d,h)` (centred XY, base on Z = 0), `cylinder(h, r, rTop?)`, `sphere(r)`, `torus(R, r)`; 2D: `rect`, `roundedRect`, `circle2d`, `ngon`, `slot`, `polygon`, `path()…`, then `.extrude(h, { twist, scaleTop })`, `.revolve()`, `.loftTo()`.
- Features: `fillet(shape, r, edgeQuery)`, `chamfer(shape, s, edgeQuery)`, `shape.shell(t)`, patterns (`linearPattern`, `circularPattern`, `mirrorCopy`).
- Placement: `attachTo`, `onFace`, `hole("top", { diameter, through, counterbore, countersink })`.
- Printing helpers: `tolerance(shape, mm)`, `chamferBottomEdges(shape, 0.4)`, `minWall(shape, mm)`.
- No imports, no file system, no `fetch` — everything is already in scope.
