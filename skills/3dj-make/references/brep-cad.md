# B-Rep CAD in 3DJ (STEP-grade parts)

Use 3DJ's parametric B-Rep engine (Replicad on OpenCascade, built in 3DJ's server kernel and in the browser workbench) instead of OpenSCAD when the person needs:

- a **STEP** file for other CAD tools, CNC, injection-moulding quotes or engineering hand-off;
- true **fillets and chamfers** on selected edges, shells, lofts, sweeps along 3D curves and profiles solved from sketch constraints;
- **assemblies** with connectors, clearance fits and standard hardware (screws, nuts, heat-set pockets, 2020 extrusion profiles) from 3DJ's `lib`;
- **mechanisms**: involute spur, ring and rack gears, belt drives, bent pipe runs, and joints that a parameter swings or slides.

## Workflow

1. Call `cad_get_starters` (free) and read `dslReference` — it is the authoritative language reference. Pass a `category` word such as `hinge`, `enclosure` or `gear` to get only the starters whose id, title or summary contain it; a word that matches nothing (`fastener` and `mating` currently match nothing) returns all of them. Start from the closest starter (project box, hinged-lid enclosure, extrusion foot, bolt flange, gripper, planetary reducer, lathe profile …) instead of a blank page.
2. Write the script: `Param.number/bool/choice` declarations first, Z up, millimetres, solids on Z = 0, then geometry, then `return` a shape, an array, or named parts `[{ name, shape, color }]`. Keep mating parts separate and seat them with connectors (`withConnectors`, then `matchTo`) or references (`withReferences`, `attachTo`). A connector's `axis` points toward the mating part. For a joint, `placeAt(target, movingName, targetName, { angle, offset })` seats the part, then turns it about the mated axis or slides it along it; make the angle a `Param` and the part swings in the viewer. `cad_get_starters` with `category: "hinge"` returns worked joints, including the hinged-lid enclosure (base, lid and pin).
3. Call `cad_validate_script` with the code and fix every reported problem. It builds the solid in 3DJ's kernel and returns `buildValidation`, `measuredGeometry` (body count, size, volume, bed fit, checks) and per-body `parts` (name, size, volume, closed). Report exactly what it returned — a build check is not an exported file and not a print inspection. Without an account it is throttled per caller address; see `tools.md`.
4. Get real files. With a 3DJ account, call `cad_export_model` with the script, the parameter values and a format chosen by where the file is going: `stl` for slicers, `step` for FreeCAD, Fusion 360, Onshape or SolidWorks, `glb` for Blender and web viewers, `zip` for an assembly (one STL per part plus a BOM). To export one body, pass `partName` exactly as the build named it in `parts`: library parts can keep their own names (a `lib.gearPair` returned `pinion z12` and `gear z30` whatever the descriptors said). It returns a `downloadUrl` and `sha256`; tell the person the link works for anyone who has it. To keep the part in their gallery (and before a print quote), call `cad_save_model` and give them its `openUrl`. Without an account, hand over the `.js` script and parameter values to paste into the CAD script editor at `https://www.3dj.app/workspace`. Either way, a file exists only once a download URL has been returned or the person has exported it.
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

```js
// Validated with cad_validate_script (solid build passed: 2 bodies, 66 × 48 × 8 mm,
// reported as "pinion z12" and "gear z30"). Ratio and centre distance: pair.ratio, pair.centerDistance.
const teeth = Param.number("Gear teeth", 30, { min: 18, max: 48, integer: true });
const pair = lib.gearPair({
  pinion: { module: 1.5, teeth: 12, faceWidth: 8, boreDiameter: 5 },
  gear: { module: 1.5, teeth, faceWidth: 8, boreDiameter: 8 },
  backlash: 0.15,
});
return [
  { name: "Pinion", shape: pair.pinion, color: "#5b6b7a" },
  { name: "Gear", shape: pair.gear, color: "#00d4ff" },
];
```

- Primitives: `box(w,d,h)` (centred XY, base on Z = 0), `cylinder(h, r, rTop?)`, `sphere(r)`, `torus(R, r)`; 2D: `rect`, `roundedRect`, `circle2d`, `ngon`, `slot`, `polygon`, `path()…`, then `.extrude(h, { twist, scaleTop })`, `.revolve()`, `.loftTo()`.
- Features: `fillet(shape, r, edgeQuery)`, `chamfer(shape, s, edgeQuery)`, `shape.shell(t)`, patterns (`linearPattern`, `circularPattern`, `mirrorCopy`).
- Placement: `attachTo`, `onFace`, `hole("top", { diameter, through, counterbore, countersink })`.
- Printing helpers: `tolerance(shape, mm)`, `chamferBottomEdges(shape, 0.4)`, `minWall(shape, mm)`.
- Mechanisms: `lib.spurGear`, `lib.gearPair`, `lib.ringGear`, `lib.rackGear`, `lib.beltDrive`, `lib.pipeRoute`; joints with `placeAt`.
- Curves: `sweep(circle2d(3), Curve.Fit([[0,0,0],[20,0,10],[40,0,0]]))` for a tube or bar, `variableSweep` to loft along a rail, `constrainedSketch()` … `.toSketch()` for a profile solved from constraints (it records whether the sketch is fully constrained).
- No imports, no file system, no `fetch` — everything is already in scope.
