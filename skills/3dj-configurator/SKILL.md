---
name: 3dj-configurator
description: Build a live 3D product configurator with 3DJ, where sliders and choices rebuild the real parametric CAD model in three.js, and show it in the conversation, share it as a link, or hand over a web page to embed. Use for "make a configurator", "let customers customise this", "interactive 3D model with sliders", "product customiser for my site", "show it live in 3D", "let me play with the dimensions", or after 3dj-make or 3dj-design when the person wants others to adjust the design themselves.
---

# 3DJ Configurator: a live model people can adjust

A configurator is a B-Rep script whose `Param.*` declarations become controls. 3DJ rebuilds the real solid in the browser on every change, so what people see is the part that would be exported and printed. It is not a picture.

Tools: `../3dj-make/references/tools.md`. Language and joints: `../3dj-make/references/brep-cad.md`.

## 1 · Write a script that configures well

Start from an existing script, or from a starter: `cad_get_starters` with `index: true`, then a `category` word.

- **3–8 knobs a customer understands.** Name them the way the customer would ("Phone width", "Lid opening"), not after internals ("wall_t2"). Give every number a `unit`, a realistic `min`/`max` and a `step`. Use `Param.choice` for discrete options (sizes, screw types, styles) and `Param.bool` for features they can switch on and off.
- **Keep the rest fixed.** Clearances, minimum walls and fillet radii that keep the part printable are derived from the knobs, not exposed. A knob must never be able to produce a part that fails the print gates. Clamp derived values with `Math.max`/`Math.min`.
- **Make motion a knob.** A hinge or slide is a `placeAt` joint with its `angle` or `offset` as a `Param`, so the part moves when the slider does.
- **Named, coloured parts:** `return [{ name, shape, color }, …]`. The configurator shows them, the exploded view separates them, and the bill of materials lists them.

Validate with `cad_validate_script` at the defaults (with `preview: true` to see the rendered isometric view inline), then at the extremes of every knob with `parameters: { ... }` set to those values (and `cad_check_assembly` when parts mate or move on `placeAt` joints). Fix every failure before going on.

## 2 · Show it live

Pick by where the person is. Say which one you used.

**A. In the conversation (3dj connector or this plugin's server).** Call `cad_get_configurator { code, parameters? }` (or `cad_validate_script { code, parameters?, preview: true }`). Where Claude supports interactive tool results, the live configurator appears under the call (3DJ's `ui://3dj/configurator` view), and the person can drag the sliders, toggle glass reference walls, animate joints, or download STL/STEP/GLB right there. Always set `preview: true` on `cad_validate_script` or call `cad_render_views` when you also want a static rendered PNG in the chat stream for hosts or mobile clients that do not expand MCP App iframes. The result carries the Param schema, parts, joints, BOM and measurements. Summarise those in a line, and do not paste the code back.

**B. A share link.** When the person wants others to use it, ask first, then call `cad_publish_configurator { code, name, parameters?, summary?, ui? }` (needs a 3DJ account). Give them `shareUrl` (`https://www.3dj.app/configurator/<id>`) and `openUrl`. Say plainly that **anyone with the link can open it and see the design**. Unpublished saved models stay private. To reopen one later, call `cad_get_configurator { modelId }`.

**C. A web page to embed or keep.** Write one self-contained HTML page. Use the import map from `cad_get_configurator`'s `cdnImports.importMap` (three.js 0.183, replicad, replicad-opencascadejs and jszip on jsdelivr) and the `<threedj-configurator>` element:

```html
<script type="importmap">{ "imports": {
  "three": "https://cdn.jsdelivr.net/npm/three@0.183.0/build/three.module.js",
  "three/": "https://cdn.jsdelivr.net/npm/three@0.183.0/",
  "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.183.0/examples/jsm/",
  "replicad": "https://cdn.jsdelivr.net/npm/replicad@1.1.0/+esm",
  "replicad-opencascadejs": "https://cdn.jsdelivr.net/npm/replicad-opencascadejs@1.1.0/+esm",
  "jszip": "https://cdn.jsdelivr.net/npm/jszip@3.10.2/+esm"
} }</script>
<script type="module" src="WEB_COMPONENT_URL"></script>
<threedj-configurator id="cfg" theme="dark" bed="220,220,250" style="display:block;height:560px"></threedj-configurator>
<script type="module">
  const cfg = document.getElementById("cfg");
  cfg.script = SCRIPT_AS_A_JS_STRING;   // or: cfg.setAttribute("model-id", "<published id>")
  cfg.addEventListener("built", (e) => console.log(e.detail.measuredGeometry));
</script>
```

`WEB_COMPONENT_URL` depends on where the page will run:

- **Claude artifact** (a published page): artifacts can load scripts only from jsdelivr's `/npm/`, unpkg and cdnjs. So use `cdnImports.webComponent`, but only after confirming the package is published: fetch `https://cdn.jsdelivr.net/npm/@neuraldeep/3dj-runtime@0.1.0/package.json`, or run `npm view @neuraldeep/3dj-runtime version`. If it isn't published yet, don't build a broken artifact. Use A or B instead, and say the artifact version needs 3DJ's runtime on npm.
- **A local file, Claude Code, or the person's own website:** `cdnImports.selfHostedWebComponent` (`https://www.3dj.app/runtime/threedj-configurator.esm.js`) works from any origin.

A built page is a file the person keeps. In the Claude app, publish it as an artifact when the runtime is on npm, and otherwise send the HTML file. In Claude Code, write it to their project and give the path.

## 3 · Hand over

- What the knobs are, with their ranges, plus any knob that animates a joint.
- Where it lives: inline, `shareUrl` (public to link holders), or the page or artifact. When there is a link, include the embed snippet: `<threedj-configurator model-id="<id>">` with the import map above.
- Files on request. Use `cad_export_model` with the chosen values: `stl`, `stl-fins` for breakaway supports, `step`, `glb` or `zip`. Export links are public to anyone with them. For print checks, use the 3dj-print skill.
- Honest limits. The live checks are geometry diagnostics, not strength or safety certification, and colours on screen are not material samples.
