# 3DJ Studio — the CAD studio of the Neuraldeep Design Studio

Design and make real, printable objects with Claude. 3DJ Studio connects Claude to [3DJ](https://www.3dj.app), Neural Deep Network Ltd's parametric CAD workbench. 3DJ builds, measures and renders every model on its own servers — so each step is something you can see, not a description, and nothing is installed or run on your machine.

```bash
/plugin install 3dj-studio --marketplace NeuralDEEPnet/3dj-claude-plugin
```

## What you can ask

| Say | Claude does |
|---|---|
| `/3dj-make` "A wall bracket that holds a 25 mm pipe 30 mm off the wall, two M4 screws, printed in PETG." | Writes a parametric model, builds and measures it in 3DJ, checks it against the 3DJ print gates, shows rendered views with thin-wall and overhang highlights, and hands over the source and (with an account) STL or STEP files with named knobs |
| `/3dj-design` "A desk headphone stand that feels premium — show me some directions first." | Brief → design DNA → three structurally different directions rendered the same way, measured, and shown with a stand-in of the headphones → critique → development → printable handoff |
| `/3dj-print` "Will this STL fit my Prusa MINI, and how should I orient it?" | Watertight, wall, overhang and bed checks, the best orientation, slicer starting points, and — on request — a 3DJ operator print quote review |
| `/3dj-configurator` "Make a configurator for this phone stand so customers can set the phone width and tilt." | Turns the parametric model into a live three.js configurator: shown right in the conversation, shared as a 3dj.app link, or handed over as a page to embed. The sliders rebuild the real CAD solid |
| `/3dj-concept` "Turn this render into a 3D model I can print: https://…/lamp.png" | Starts a 3DJ image-to-3D (or text-to-3D) generation, follows it to the finished mesh and hands over the file with a workbench link |

Skills also start on their own when a request matches.

## What's inside

| Component | Purpose |
|---|---|
| `3dj` MCP server | The 3DJ service (`https://www.3dj.app/api/mcp`, 32 tools): OpenSCAD and B-Rep CAD starters, script validation with rendered views, assembly interference and joint checks, revision comparisons, live configurators, STL/STEP/GLB export, A4 drawings (PDF/SVG/DXF), gallery saves with revision history, STEP/STL/GLB import as reference geometry, a private feature library and review comments, print quote requests, text- and image-to-3D generation |
| `3dj-make` skill | Functional printable parts; holds the shared references (3DJ tools, B-Rep CAD, OpenSCAD rules, print gates, form cookbook) |
| `3dj-design` skill | The studio flow with compared directions |
| `3dj-print` skill | Print-readiness, orientation and operator quote reviews |
| `3dj-configurator` skill | Live three.js product configurators from a parametric model: inline, share link or embeddable page |
| `3dj-concept` skill | Organic and sculptural concept meshes from a text prompt or an image (3DJ account) |
| `3dj-critic` agent | Independent review of rendered models against the brief and design DNA |

## Free versus account

- **Free, no account**: 3DJ's public tools — OpenSCAD and B-Rep starters, building and measuring B-Rep scripts with rendered views, print-check highlights, assembly checks and revision comparisons (throttled per conversation), and the evidence schema.
- **With a 3DJ account** (OAuth sign-in): hosted OpenSCAD compile (20 per hour, 100 per day), real STL, STEP and GLB exports of B-Rep parts (30 per hour), saving parts to your gallery, text- and image-to-3D generation (hourly and daily allowance), your downloads, and print quote requests.

Sign in when a skill asks: in Claude Code run `/mcp`, choose **3dj**, then **Authenticate**; in the Claude app use Settings → Connectors → 3dj. Never paste keys into chat.

## The checks

Every check follows the 3DJ Design Evidence Pack v1 gates: watertight, 0.8 mm nozzle-safe and 1.2 mm recommended walls, 45° overhang, 220 × 220 × 250 mm bed. They are geometry screens, not manufacturing certification.

## What this plugin runs, sends and fetches

- **Runs locally**: nothing. The plugin is skills, an agent and the 3DJ MCP server address; it ships no scripts or programs.
- **Sends to 3DJ, only when a 3DJ tool is called**: the OpenSCAD or CAD script source and parameters of each model the skills build, measure, render, export or save (every build and picture runs on 3DJ's servers), the model ids you ask about, the STEP, STL or GLB files (or their public URLs) you ask it to import, which 3DJ keeps privately until you delete them, review comments and saved features you ask it to store, the prompts or public image URLs you ask it to turn into 3D, and, for a quote request, the model, dimensions, material, colour, quantity, intended use and destination country and region (never a street address or payment details). For text-to-3D, 3DJ first creates one AI reference image from your prompt and reconstructs the mesh from it; generated models go to your 3DJ gallery. Exported files are stored by 3DJ and their download links work for anyone who has them; saved parts go to your gallery; a configurator you ask it to publish can be opened by anyone with its link. A configurator page loads three.js, replicad and the 3DJ runtime from jsDelivr or www.3dj.app in the viewer's browser. 3DJ's handling of that data is described in its [privacy policy](https://www.3dj.app/privacy).
- **No telemetry**, no credentials read from your machine, and no other network destinations.

## Privacy

Data sent to the 3DJ service is covered by the [3DJ privacy policy](https://www.3dj.app/privacy) (Neural Deep Network Ltd, 3d@neuraldeep.net). 3DJ refuses credentials and sensitive personal data in tool arguments.

## Works with the Design Studio

Install alongside `neuraldeep-design-studio` from the [Neuraldeep Design Studio marketplace](https://github.com/NeuralDEEPnet/claude-design-skills). Its `3d-object-design` and `design-director` skills run the full studio process; 3DJ Studio supplies the modelling, comparison and print stages.

## Boundaries

Quote tools create review requests and record acceptance of operator-issued terms only, after you confirm: no payment, print booking or shipping happens through this plugin. Bench and 3DJ checks do not certify strength, safety or manufacturability.

Neural Deep Network Ltd · [3dj.app](https://www.3dj.app) · [support](https://www.3dj.app/support) · [privacy](https://www.3dj.app/privacy) · [terms](https://www.3dj.app/terms) · MIT licence
