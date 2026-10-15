---
layout: post
title: "Check atlas frame numbers before tuning an exported effect"
description: "An asset-backed four-cell atlas example with an actual export manifest and explicit support warnings."
date: 2026-10-15
---

A particle that shows the wrong part of a texture can look like a timing bug. It may instead be a frame-selection problem, an incorrect grid, or a provider resolving the wrong asset. A small diagnostic atlas makes those possibilities easier to separate than a sheet of similar-looking sparks.

This example uses a generated 64-by-64 PNG containing four solid-color quadrants. The source file is `assets/quadrants.png`; every cell is 32 pixels square. The top row is red then green, and the bottom row is blue then yellow. The colors are diagnostic markers, not finished effect art.

The export and frame-sampler observations below were checked with nixie-fx 0.1.20. No screenshot comparison or GPU rendering result is claimed.

## Keep the source and runtime paths distinct

The authoring project stores its effect under `particle-data/effects/atlas-frame-check.json`. Its project settings use `assets` as the asset root and `out/vfx` as the export directory. Consequently, the authored texture reference is `quadrants.png`, relative to that root. It is not the absolute path of a file on the author's computer.

The completed export contains a manifest, compiled effect JSON, and the copied PNG. In this particular bundle the manifest records the following entries. This is a reduced excerpt from the generated manifest, not a replacement manifest to paste over a real export.

```json
{
  "effects": [{
    "id": "atlas-frame-check",
    "path": "effects/atlas-frame-check.json"
  }],
  "assets": [{
    "id": "quadrants",
    "type": "texture",
    "path": "quadrants.png"
  }]
}
```

The runtime should resolve asset paths from the bundle root. A host serving this directory at `/vfx` would load `/vfx/quadrants.png`, not `/vfx/assets/quadrants.png`. Do not invent a second folder convention after export. Preserve the structure actually described by the manifest.

## Make the grid impossible to misread

Enable texture-sheet animation on one billboard emitter. Use a two-by-two grid, start frame zero, end frame three, one cycle, and no random starting frame. Begin with a fixed frame rather than animation. The simplest question is whether each requested cell is the expected cell.

The frame sampler in the installed package returned these coordinates when given fixed frame values zero through three:

| Frame | Column | Row | Diagnostic cell |
| --- | --- | --- | --- |
| 0 | 0 | 0 | Red, top left |
| 1 | 1 | 0 | Green, top right |
| 2 | 0 | 1 | Blue, bottom left |
| 3 | 1 | 1 | Yellow, bottom right |

The coordinates are a verified sampler result. The color association describes the generated image layout. A rendered check is still required to confirm the texture provider and renderer's UV handling together. If a rendered frame appears vertically flipped, inspect that path before changing the authored frame order to compensate.

## Use frame values, then verify the curve

In this check, setting `frameOverTime` to constant values zero, one, two, and three selected the corresponding frames. Treating every input as a normalized zero-to-one position did not produce four distinct frames. That is why a fixed-frame probe belongs before a timing adjustment.

```js
import {
  createDefaultParticleEffect,
  createConstantParticleScalar as scalar,
  sampleTextureSheetAnimationFrame
} from 'nixie-fx';

const effect = createDefaultParticleEffect();
const sheet = effect.emitters[0].advanced.textureSheetAnimation;
Object.assign(sheet, {
  tiles: [2, 2], startFrame: 0, endFrame: 3,
  cycles: 1, randomStartFrame: false
});
for (const frame of [0, 1, 2, 3]) {
  sheet.frameOverTime = scalar(frame);
  console.log(sampleTextureSheetAnimationFrame(sheet, 0.5, 13));
}
```

After the fixed-cell checks pass, add the intended frame-over-time curve and sample several known ages. Record the final-frame behavior rather than assuming the end of the lifetime stays on the last cell. Keep random starts disabled until the deterministic sequence is understood.

## Read a successful export's warnings

The local export completed with one effect and no validation errors. It also reported partial support for texture-sheet animation on both backends. The manifest's overall support status was `partial`, and `portableSubset` was false despite the portable target profile. Export success therefore did not establish that every authored subfield had full runtime support.

The [NixieFX particle editor for PixiJS and Three.js reference](https://nixiefx.com/vfx-runtime-docs/) explains the bundle and support-report boundary. Retain those diagnostics with the example. A target profile expresses an intention to validate; it is not a substitute for reading the result.

## Finish with a renderer check

Load the exported bundle into the actual PixiJS or Three.js integration, require the intended backend, and preload the referenced texture through the appropriate provider. Render fixed frames before the animated sequence. Check all four colors, the absence of neighboring-cell leakage, and the expected lifetime progression. Repeat after changing filtering, texture size, or export settings.

This example uses a regular grid. It does not establish support for arbitrary packed, rotated, or trimmed atlas rectangles. The PixiJS depth notes also describe ordering semantics rather than GPU depth behavior. Keep those limits separate from the frame-number test, and leave visual acceptance pending until the exported asset is inspected in the destination renderer.
