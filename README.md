# Three.js Object Sculptor (Claude Code)

Turn the object in an attached image into a quality-gated, animation-ready procedural Three.js model built entirely with code.

Three.js Object Sculptor is a **Claude Code plugin** for rebuilding the visible object in a user-provided image as a code-only Three.js model. It does not do photogrammetry, download an art pack, or extract a perfect mesh from one image. Instead, it guides Claude through a sculpting workflow: validate the image, describe the object precisely, decompose it into geometry and material systems, build from blockout to detail, wire an animation-friendly hierarchy, then compare the browser render against the original reference.

> This is the Claude Code port of the original Codex plugin. The workflow, skill, and Python helper scripts are the same; only the plugin manifest, install path, and screenshot/vision tooling references are adapted for Claude Code.

## At A Glance

- **Name:** Three.js Object Sculptor
- **Runtime:** Claude Code plugin (skill + slash command + Python helper scripts)
- **Input:** an attached object image, reference screenshot, or local image path
- **Output:** a code-only procedural Three.js object factory, backed by an `ObjectSculptSpec`
- **Primary goal:** recreate the target object's silhouette, component structure, materials, lighting response, and action-ready hierarchy in browser-friendly Three.js code
- **Best for:** animation-ready real-time props, game objects, scene dressing, destructible objects, product-style objects, botanical objects, mechanical parts, and stylized reference reconstructions
- **Not for:** photogrammetry, exact mesh extraction, scanned assets, downloaded art packs, or guaranteed production-perfect geometry from one image

## What It Does

- Validates whether an image is suitable for procedural 3D reconstruction.
- Creates a pre-spec complexity assessment before code generation.
- Writes an `ObjectSculptSpec` with component hierarchy, materials, lighting, pivots, sockets, animation anchors, destruction anchors, and quality targets.
- Enforces a staged build pipeline: blockout, structural pass, form refinement, material pass, surface pass, lighting pass, interaction pass, and optimization.
- Generates a code-only Three.js factory skeleton from the current unlocked sculpt pass.
- Designs the generated object as an action-ready hierarchy, so later animation, transformation, physics, or destruction requests have real pivots and attachment points to use.
- Packages reference/render screenshots into one comparison sheet for vision review.
- Records self-correction reviews with overall, layer, and critical feature scores.
- Supports reference-derived procedural PBR evidence: albedo, roughness estimate, height, normal, and AO maps.

## Requirements

- Claude Code with plugin support.
- Python 3.10 or newer (for the helper scripts).
- A browser project using Three.js when you want to implement the generated factory.
- For visual acceptance: a screenshot from the rendered model. Claude reads the screenshot directly with the Read tool. A browser MCP (e.g. Playwright MCP) is optional and only used if already configured or explicitly requested.

The helper scripts use Python standard-library modules and shell image tooling when available. They do not require Playwright or a downloaded Chromium bundle.

## Install For Claude Code

### Option A — local marketplace (recommended for development)

From the directory that contains this plugin folder, add it as a local marketplace and install:

```bash
# Point Claude Code at the folder containing .claude-plugin/marketplace.json
/plugin marketplace add ./Three.js-Object-Sculptor-Claude-Code-Plugin
/plugin install threejs-object-sculptor@threejs-object-sculptor-marketplace
```

You can also run these from the `/plugin` interactive menu inside Claude Code.

### Option B — from GitHub (recommended for everyone else)

Inside a Claude Code session:

```bash
/plugin marketplace add martinstefanovic/three.js-object-sculptor-claude-code-plugin
/plugin install threejs-object-sculptor@threejs-object-sculptor-marketplace
```

`/plugin marketplace add` reads `.claude-plugin/marketplace.json` from the repo root; the
second line installs the plugin by its **id** (`threejs-object-sculptor`), not the repo name.

After installing, run `/reload-plugins` (or start a new Claude Code session) so the skill and
the `/threejs-object-sculptor:sculpt` command load.

## Quick Start

Attach or reference an object image and either invoke the skill directly:

```text
Use the object-to-threejs-procedural skill to turn the object in ./reference/oak-tree.png
into a procedural Three.js model built entirely with code.
```

…or use the slash command:

```text
/threejs-object-sculptor:sculpt ./reference/oak-tree.png — real-time browser prop, action-ready for animation
```

You can attach an image directly in the message instead of passing a path.

For best results, include the intended use (standalone model, game prop, hero render, playable object, destructible object, or animation rig).

The plugin will guide Claude through:

1. Image suitability check.
2. Pre-spec complexity and quality contract.
3. Detailed object sculpt spec.
4. Strict validation.
5. Pass-by-pass Three.js factory generation.
6. Browser screenshot review.
7. Vision comparison and self-correction.

## Recommended Workflow

Run the scripts from the plugin root.

Probe a reference image:

```bash
python3 scripts/probe_reference_image.py ./reference/oak-tree.png
```

Create a pre-spec assessment:

```bash
python3 scripts/new_pre_spec_assessment.py "Ancient Autumn Oak" \
  --image ./reference/oak-tree.png \
  --complexity complex \
  --out assessment.json
```

Create a starter sculpt spec:

```bash
python3 scripts/new_sculpt_spec.py "Ancient Autumn Oak" \
  --image ./reference/oak-tree.png \
  --assessment assessment.json \
  --out object-sculpt-spec.json
```

Validate the spec:

```bash
python3 scripts/validate_sculpt_spec.py object-sculpt-spec.json --strict-quality
```

Check which sculpt pass is unlocked:

```bash
python3 scripts/sculpt_pass_orchestrator.py status object-sculpt-spec.json
```

Generate the current pass:

```bash
python3 scripts/generate_threejs_factory.py object-sculpt-spec.json \
  --out src/createObjectModel.ts
```

Create a comparison sheet after rendering the model:

```bash
python3 scripts/make_visual_comparison_sheet.py \
  --reference ./reference/oak-tree.png \
  --render ./screenshots/oak-render.png \
  --out ./screenshots/oak-comparison.png \
  --json
```

Then read `oak-comparison.png` with the Read tool and score it, and record the review:

```bash
python3 scripts/append_sculpt_review.py object-sculpt-spec.json \
  --pass-id blockout \
  --fidelity 0.82 \
  --action continue \
  --summary "Blockout silhouette and primary trunk fork are acceptable." \
  --render-screenshot ./screenshots/oak-render.png \
  --comparison-image ./screenshots/oak-comparison.png \
  --ai-vision-score 0.82 \
  --ai-vision-notes "Main proportions pass; canopy microstructure remains deferred." \
  --in-place
```

Sync the pass state:

```bash
python3 scripts/sculpt_pass_orchestrator.py sync object-sculpt-spec.json --in-place
```

## Quality Gates

The plugin uses two levels of visual acceptance:

- **Overall match:** silhouette, proportions, camera/view, material read, and lighting.
- **Semantic feature match:** selected critical object features scored from the same full reference/render comparison image.

If a critical feature fails its threshold, the pass fails even when the global score is high. Claude inspects the comparison sheet with its own vision (via the Read tool) — the scripts only package evidence, they do not judge visual quality.

## Example Output

A worked example lives in [`examples/offroad-suv/`](examples/offroad-suv/) — a stylized low-poly
off-road SUV, generated end-to-end with this skill from a single reference image. It is a
self-contained Three.js scene with an action-ready wheel hierarchy, GTAO + bloom post-processing,
real headlight/roof spotlights, and a day/night ("off-road") mode toggle (press `N`).

To view it (ES-module scenes must be served over HTTP, not opened as a `file://`):

```bash
cd examples/offroad-suv
python3 -m http.server 8781
# then open http://localhost:8781/
```

## Project Layout

```text
.claude-plugin/plugin.json            # Claude Code plugin manifest
.claude-plugin/marketplace.json       # local/Git marketplace entry
commands/sculpt.md                    # /threejs-object-sculptor:sculpt slash command
skills/object-to-threejs-procedural/  # the skill (SKILL.md + references/)
scripts/                              # Python helper scripts
examples/offroad-suv/                 # worked example scene
```

Important scripts:

- `probe_reference_image.py`: technical image metadata probe.
- `new_pre_spec_assessment.py`: complexity and quality-contract skeleton.
- `new_sculpt_spec.py`: starter `ObjectSculptSpec`.
- `validate_sculpt_spec.py`: structural and strict quality validation.
- `sculpt_pass_orchestrator.py`: pass locking and pipeline sync.
- `generate_threejs_factory.py`: current-pass Three.js factory generator.
- `make_visual_comparison_sheet.py`: full reference/render comparison image.
- `append_sculpt_review.py`: self-correction review recorder.
- `extract_reference_pbr.py`: reference-derived PBR evidence extraction.

## Limitations

- A single image cannot reveal hidden sides or guarantee exact geometry.
- Transparent glass, smoke, liquid, fur, fine cloth, and exact likeness tasks may require extra references or a lower-fidelity target.
- The generated factory is a starting point for procedural construction, not a finished asset pipeline replacement.
- Vision review is expected for acceptance; the scripts package evidence but do not judge visual quality by themselves.

## License

MIT
