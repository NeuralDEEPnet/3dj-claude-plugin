# Maintaining 3DJ Studio

This repository is the plugin: the repository root is the plugin folder that the Claude directory installs, so everything committed here ships to users. Keep it to the plugin's own files.

## Layout

| Path | What it is |
|---|---|
| `.claude-plugin/plugin.json` | Manifest. Raise `version` with every release |
| `.claude-plugin/marketplace.json` | Lets people install straight from this repository |
| `.mcp.json` | The 3DJ MCP server. Must point at the same URL as 3DJ's connector listing |
| `skills/3dj-make/` | Functional parts, plus the shared references |
| `skills/3dj-design/`, `skills/3dj-print/`, `skills/3dj-concept/` | Studio flow, print-readiness and generated concept meshes; they use the references in `3dj-make` |
| `agents/3dj-critic.md` | Independent review agent |
| `evals/` | Graded cases and trigger queries for `claude plugin eval` |

## Before every release

```bash
claude plugin validate --profile directory --strict .
```

Every pattern in `skills/3dj-make/references/form-cookbook.md` must compile watertight with 3DJ's `openscad_compile_model` and pass the wall, overhang and bed gates; extract the `openscad` blocks and compile each one after editing the cookbook. B-Rep examples in `brep-cad.md` must pass `cad_validate_script`.

## Rules that keep the directory listing healthy

- **Disclose behaviour.** The README's "What this plugin runs, sends and fetches" section must stay complete and true. Any new network destination, download or data sent to 3DJ goes there first.
- **No scripts or programs.** The directory accepts scripts and files outside its allowed types only from Verified organizations, so the plugin ships none: every build, measurement and render runs on 3DJ's server through its MCP tools.
- **No install or download-and-run commands**, even in documentation, and no copied script tags that load remote code. The directory holds or refuses such versions from organizations that aren't Verified. Point people to [3dj.app/plugins](https://www.3dj.app/plugins) for setup, and let skills take script addresses from 3DJ's tool results at run time.
- **`plugin.json` holds only fields in the [manifest reference](https://code.claude.com/docs/en/plugins/manifest-reference)**, plus the listing fields the directory reads (`privacyPolicyUrl`, `icon`). Anything else is reported as an unrecognized field.
- **No credentials in files**, no reading tokens from the user's environment, no package-manager config files (`.npmrc` and similar).
- **Files** stay Markdown, text, JSON, YAML, or PNG/JPEG/GIF/WebP/ICO images, UTF-8 with no control characters, each under 256 KiB (images excepted), 512 files at most. No `.DS_Store`, symlinks, submodules or LFS.
- **Skills describe; they don't steer.** Skills must not tell Claude to call connectors the person didn't ask for, and must not tell it to follow instructions fetched at runtime. Server-provided language notes are reference data.
- **One service, two routes.** Skills name only this plugin's `3dj` server and the Claude app's 3dj connector. No other connector, gateway or proxy tool, and no pointers to install other plugins or marketplaces: the directory reads those as steering, and the README promises no other network destinations.
- **Generation stays disclosed.** The `3dj-concept` skill tells people before the first generation that text-to-3D creates an AI reference image on 3DJ's servers, and the README's disclosure section says the same. Keep both whenever generation changes.
- **Honest claims.** 3DJ results are geometry screens, never manufacturing, structural or safety certification.

## The 3DJ server

The MCP server lives in the 3DJ application, not here. When its tools change, update `skills/3dj-make/references/tools.md` and any skill steps that name the tools, then validate and release.
