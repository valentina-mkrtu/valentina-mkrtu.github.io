---
layout: post
title: "A character aura that follows the same rules in PixiJS and Three.js"
description: "A small local-space aura recipe with explicit coordinate mapping, pinned runtime versions, and separate adapter controls."
date: 2026-10-14
---

A character aura should communicate attachment. When the character moves, the ring belongs with the character. That makes it a useful test of a particle effect’s simulation space: a world-space wake may look attractive while conveying the wrong gameplay state.

This exercise uses nixie-fx 0.1.20, PixiJS 8.19.0, and Three.js 0.184.0. It starts with an asset-free procedural circle so that missing textures cannot conceal a coordinate or lifecycle problem. The aim is a comparable aura in two renderers, with differences made explicit. It does not claim that every material or rendering setting is portable.

## Define the visual contract

The aura is a narrow ring of cyan particles around a moving origin. New particles appear continuously and fade over a short lifetime. Already emitted particles stay attached to the origin. A stop command removes the aura, and a new activation begins a fresh run.

For review, keep the camera fixed and move only the origin. This isolates attachment from camera movement. Use the same seed and particle parameters in both adapters. Then compare the shape and timing before adding a character sprite, a 3D model, or game-state logic.

The [NixieFX particle editor for PixiJS and Three.js](https://nixiefx.com/pixijs-particle-effects/) is the authoring context for this workflow. Here, the package’s default definition supplies a reproducible starting point directly in code. A production pipeline should validate and export its authored effect and inspect the support report for each target backend.

## Create the aura definition

Install the pinned packages in a browser application with a bundler. Put this function in `aura.js`. It creates a fresh definition so that editing one test case cannot mutate another.

```js
import {
  createDefaultParticleEffect,
  createConstantParticleScalar as scalar,
} from 'nixie-fx';

export function makeAura() {
  const effect = createDefaultParticleEffect('aura', 'Character aura');
  effect.targetProfile = 'portable';
  effect.timeline.duration = 4;
  effect.timeline.loop = { enabled: true, start: 0, end: 4 };
  const e = effect.emitters[0];
  e.duration = 4;
  e.loop = true;
  e.maxParticles = 128;
  e.spawn.shape = 'circle';
  e.spawn.radius = 0.8;
  e.spawn.radiusValue = scalar(0.8);
  e.spawn.radiusThickness = 0;
  e.spawn.simulationSpace = 'local';
  e.spawn.rate = 48;
  e.spawn.rateValue = scalar(48);
  e.initializeParticle.lifetime = scalar(0.6);
  e.initializeParticle.size = scalar(0.12);
  e.initializeParticle.velocity.speed = scalar(0.05);
  e.initializeParticle.color.color = [0.25, 0.7, 1, 1];
  e.render.blend = 'additive';
  e.modules.color = true;
  e.color.gradient.alphaStops = [
    { position: 0, alpha: 1 },
    { position: 1, alpha: 0 },
  ];
  return effect;
}
```

The radius uses simulation units rather than screen pixels. The particle cap is a bound for this example, not a recommended budget for every game. Increase one parameter at a time when evaluating a different look.

## Map the origin into each scene

In Three.js, create a `ThreeVfxRenderer` with your scene and camera, then call `createEffect(makeAura(), { seed: 42 })`. Update the instance with `setTransform({ position: [x, y, z] })`. A camera looking toward the XY plane makes the ring easy to compare with the 2D view.

In PixiJS, mount a `PixiVfxRenderer` under a dedicated container. Center that container in the view and use a deliberate scale, such as 65 pixels per simulation unit. In this comparison, a negative Y scale maps positive simulation Y upward. Call `createEffect(makeAura(), { seed: 42 })`, then move the instance with `setPosition([x, y, z])`.

Do not move both the parent and the instance by the full character offset. That applies the translation twice. Choose one ownership model and document it. A parent can handle the fixed screen mapping while the instance handles character movement.

## Advance each simulation once

Both renderer adapters accept elapsed seconds in `update`. A host callback that reports milliseconds needs division by 1,000. If the renderer owns the instance, do not also update the instance separately.

For this comparison, the browser frame loop advances each renderer once and then draws each canvas. The host limits a very long frame step to avoid a large jump after a suspended tab. That changes elapsed simulation time during stalls, so it should be a conscious demo choice, not an invisible production assumption.

## Review attachment and lifecycle separately

First move the origin left and right while the aura runs. The local-space ring should remain attached. Then change only simulation space to world and repeat: previously emitted particles should reveal the difference. Return to local space for the final aura.

Next pause and resume. In this package version, Three.js exposes instance `pause()` and `play()`. PixiJS exposes different instance methods; the host can pause this demo by withholding simulation updates while continuing to render. That freezes every instance under the paused renderer. It is not an independent per-instance pause API.

Restart also differs: use `restart()` on the Three.js instance and `reset({ seed: 42 })` on the PixiJS instance. Stop and removal should be checked independently. Remove completed instances from their owning renderer instead of leaving an ever-growing collection of invisible objects.

## Limits and verification

The recipe intentionally avoids prepared 3D meshes, mesh-surface emission, lighting, node materials, trails, and bloom. PixiJS does not provide the same world-depth and lighting semantics as Three.js. A portable profile is a validation target, not a promise of identical pixels from different cameras and render pipelines.

The accompanying local lab rendered the procedural recipe in both adapters. Its Three.js checks covered pause, resume, stop, restart, and removal. These checks do not establish a mobile frame budget or verify every NixieFX feature. Re-run visual checks after replacing the recipe, changing package versions, or introducing textures and materials.

AI disclosure: AI assisted with this article and the local verification code.
