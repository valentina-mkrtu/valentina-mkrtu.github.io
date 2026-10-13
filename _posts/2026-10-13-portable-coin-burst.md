---
layout: post
title: 'A coin burst that keeps its timing across PixiJS and Three.js'
description: 'An authoring and integration recipe for a small portable coin burst, with separate checks for simulation and rendering.'
date: 2026-10-13
published: true
---

AI disclosure: This article was generated with AI. The values below are proposed authoring settings, not measured results or a completed visual test.

## Define the cue before choosing a renderer

A reward burst should make the claim event visible without hiding the balance that changed. Begin with a small one-shot design: a dozen coin billboards spread outward from the reward origin and disappear within roughly three quarters of a second. Keep the actual balance update in game logic. The particle effect communicates the event and must not determine whether the reward is granted.

The same cue may appear over a PixiJS interface and above an object in a Three.js scene. Shared timing is useful, but the two views do not have the same coordinate system, camera, or depth behavior. Treat those differences as integration work instead of assuming that identical effect data guarantees identical pixels.

## Author a portable source

Create a NixieFX project and choose the portable target profile. Use one billboard emitter, a point origin, a single initial burst, and no looping. Start with twelve particles, lifetimes between 0.5 and 0.75 seconds, a modest outward velocity, and a warm coin color. Give the particles a short readable middle phase before fading their alpha to zero.

Those values are a starting recipe. Adjust size and speed together while checking the actual camera framing. A velocity that reads clearly in a 2D overlay may travel too far when interpreted in your 3D world's units.

Use a coin texture you own or are permitted to distribute. For an initial timing check, a procedural circle can stand in for the coin silhouette, but label that preview honestly. It does not demonstrate the final texture, edge quality, or orientation.

The [NixieFX particle editor for PixiJS and Three.js](https://nixiefx.com/pixijs-particle-effects/) describes the authoring and Pixi integration path. Keep this recipe within the portable feature set instead of adding a Three.js-only setting and expecting the 2D output to match it.

## Export the asset with its support reports

Validate the project and export the runtime bundle. Deliver the manifest, compiled effect files, and every referenced asset together. The authoring JSON remains editable source; the game should consume the exported bundle.

Inspect both backend reports. Resolve blocked settings and review any partial support warning before calling the effect portable. Do not hide a warning just because one viewport appears plausible. The report can identify a difference that is difficult to notice in a small preview.

Keep the package versions and the exported effect identifier with the recipe. An integration example should select the actual ID from the bundle rather than assume the filename is the ID.

## Integrate one trigger in each host

In PixiJS, load the bundle for the pixi2d backend, preload the coin texture, and create a PixiVfxRenderer on a dedicated effects container. Convert the reward origin into that container's coordinates. Spawn once when the game confirms the claim, then advance the renderer once per ticker frame using seconds.

In Three.js, load the same export for the three3d backend and create a ThreeVfxRenderer with the scene and active camera. Place the origin in the intended world position. Use the same seed when comparing motion, and advance this renderer once per host frame in seconds as well.

Do not update both the renderer and its owned instance in the same frame. That would advance the simulation twice. When the one-shot completes, remove its instance through the owning renderer. On scene teardown, destroy the renderer and release provider-owned textures according to their ownership rules.

## Compare the same moment

Use the following review fixture instead of judging two unrelated plays:

```text
Effect: exported coin-burst ID from the manifest
Seed: 269
Start: one confirmed reward event
Comparison times: 0.10 s, 0.35 s, 0.70 s
Backgrounds: one light and one dark
Cameras: fixed for timing comparison, then moving for integration review
Record: runtime versions, export revision, scale mapping, captures, warnings
```

First compare timing with fixed cameras. Then move the camera and test the coordinate mapping. In the 2D overlay, the effect should remain attached to the intended UI location. In the 3D scene, confirm that the reward appears where the player expects and that foreground geometry interacts with the particles as designed.

Capture the three checkpoints from actual runtime output. This article provides no screenshots and claims no completed comparison. If you only have the editor preview, leave the runtime review pending.

## Check the limits before shipping

PixiJS does not implement Three.js GPU depth testing, lit particle shading, or mesh-surface emission in the same way. Avoid lit shading and mesh-surface emission for this portable billboard recipe. Prepared 3D coin meshes would require a separate design decision and cannot be treated as identical 2D coins. Editor preview bloom also needs an explicit game rendering setup if you want that appearance at runtime.

Use repeated claims to check instance cleanup, then test the busiest intended reward state on a representative device. Twelve particles is a design input, not a performance guarantee. Review texture coverage and transparency as well as particle count.

The deliverable is complete when both integrations load the same export, a single event produces a single burst, the required resources load successfully, and reviewers have recorded the observed differences. Matching intent across renderers is a useful goal. Claiming parity without looking at both outputs is not a substitute for that review.
