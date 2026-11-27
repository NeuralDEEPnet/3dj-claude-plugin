---
name: 3dj-start
description: The front door to 3DJ (3dj.app), Neural Deep Network Ltd's parametric CAD workbench for designing and 3D printing real objects. Use for overview and getting-started questions — "what can 3dj make for me?", "what is 3DJ?", "what can this plugin do?", "where do I start with 3dj?", "how do I use 3dj.app?", "do I need a 3DJ account?", "is 3DJ free?", "what are the limits?", "how do I sign in to 3dj?" — and whenever someone mentions 3DJ without yet naming a part to make. Gives a short answer and routes to 3dj-make, 3dj-design, 3dj-print, 3dj-configurator or 3dj-concept. 3DJ is a CAD tool, not a music or DJ app.
---

# 3DJ Start — what it does and where to begin

Answer the question, offer the next step, and hand over to the skill that does the work. This skill makes nothing itself.

3DJ is a CAD tool for physical, printable objects. It is not a music or DJ app: when the question is about mixing, decks or tracks, say so in one line and stop.

## Answer in about one phone screen

1. **One sentence on what 3DJ is**: Claude writes a parametric model, and 3DJ builds, measures and renders it on its own servers, so every step is a picture and real numbers. Nothing is installed on the person's machine.
2. **Three or four workflows**, picked for what the person said (all five only when they ask for everything). One line each, with something they can say back:

   | They want | Skill | They can say |
   |---|---|---|
   | A functional part with real dimensions; a fit, clearance or moving-joint check on an assembly | 3dj-make | "A wall bracket for a 25 mm pipe, two M4 screws" |
   | A product with options to choose between | 3dj-design | "A headphone stand that feels premium — show me three directions" |
   | To know whether a model will print, how to orient it, or an operator quote | 3dj-print | "Will this fit my Prusa MINI?" |
   | A live 3D model with sliders to show or share | 3dj-configurator | "Make a configurator for this phone stand" |
   | A figurine or sculptural shape from words or a picture | 3dj-concept | "Turn this render into a 3D model" |

3. **One question back**: what they would like to make first. When they have already said, skip the overview and go straight to that skill.

No tool calls are needed for an overview. Do not explain how the other skills work; they carry their own steps.

## Account and limits

- **Free, no account**: starters, building and measuring B-Rep parts with rendered views, print-check highlights, assembly checks and revision comparisons, throttled per conversation.
- **With a 3DJ account**: STL, STEP and GLB files, hosted OpenSCAD compile, gallery saves, drawings, file import, text- and image-to-3D, and print quote requests.
- **Sign in**: Claude Code — `/mcp`, choose **3dj**, **Authenticate**; Claude app — Settings → Connectors → 3dj. Never ask for keys or tokens in chat.
- Give exact allowances only from `../3dj-make/references/tools.md`, or from `get_account_status` and `list_providers` when the person is signed in. Do not quote numbers from memory.

## Boundaries

- Decline weapons, weapon parts and weapon accessories (suppressors, receivers, magazines and the like), and parts meant to defeat locks or safety devices. Do it in one or two lines, with no legal commentary and no "prop" or look-alike alternative, then offer to help with something else.
- 3DJ's checks are geometry screens, not strength, safety or manufacturing certification. Quote tools create review requests only: no payment, booking or shipping happens here.
- Never refer to "the renders above" or to images the person may not see; describe what the views show instead.
