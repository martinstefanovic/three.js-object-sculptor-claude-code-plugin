---
description: Turn an object reference image into a quality-gated, action-ready procedural Three.js model.
argument-hint: [image path or description of the object + intended use]
---

Use the `object-to-threejs-procedural` skill to reconstruct the object from the reference image as a code-only Three.js model.

Reference image / request: $ARGUMENTS

Follow the skill's workflow:

1. Run the image suitability gate and return a verdict with scores.
2. Run the Pre-Spec Assessment Gate and write the quality contract.
3. Author (or revise) the `ObjectSculptSpec`, then validate it with `scripts/validate_sculpt_spec.py --strict-quality`.
4. Generate the current unlocked sculpt pass with `scripts/generate_threejs_factory.py`.
5. Render in the browser, capture a screenshot, build one comparison sheet, and review it with your own vision (read the image with the Read tool).
6. Record the self-correction review and only advance passes when the gate allows.

If no image is provided, ask for one. If the intended use is missing, assume a real-time browser Three.js prop.
