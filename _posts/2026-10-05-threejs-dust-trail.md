---
layout: post
title: "Build a Three.js dust trail that stays behind the character"
description: "A NixieFX dust-trail recipe with world-space placement, a worked movement example, and explicit runtime verification steps."
date: 2026-10-05
---

A dust trail should help a player read movement without hiding the character's feet. For a first implementation, keep the scene deliberately plain: a floor, a moving marker, a fixed camera, and one dust emitter. That makes it easier to distinguish particle problems from camera motion, lighting, or animation.

This is an implementation recipe, not a tested demo. The numerical values below are proposed starting points. An exported effect and a browser verification pass are still needed before claiming that this particular result works.

## Decide what the trail should communicate

Use a straight six-unit path for the first test. Move the marker at three units per second, stop it, and watch the remaining dust fade. Add a right-angle turn only after the straight movement is readable.

Three observations matter: new dust appears near the contact point, old dust stays behind, and standing still does not build an opaque cloud. Write those down before tuning the artwork. They give the reviewer something more precise than “looks dusty.”

A three js particle editor can help tune individual puffs, but the game still determines where and when the effect moves. Keep that responsibility visible in the implementation.

## Author one restrained emitter

Create a NixieFX project in the editor, then create a “Dust Trail” effect targeting `three-world-3d`. Start with a single looping billboard emitter in world simulation space. Use a procedural soft circle, unlit shading, and alpha blending. Leave texture files, material graphs, sub-emitters, and decorative particle trails out of this first version.

Try these art-direction settings, adjusting their scale to your scene:

- A narrow spawn area centered just above the ground.
- Particle lifetime around 0.45 seconds.
- Initial size around 0.12 scene units, expanding modestly over life.
- Muted brown color, with opacity falling to zero.
- Small upward motion and slight horizontal variation.
- Emission over distance at roughly eight particles per scene unit, with continuous time-based emission and scheduled bursts disabled.

These are tuning suggestions, not a supplied preset or benchmark. Save the effect source alongside the host application so another developer can inspect the actual settings you chose.

## Work through the numbers

At three units per second and eight particles per unit, the intended emission density corresponds to roughly 24 particles per second during steady travel. With a 0.45-second lifetime, a rough steady-state estimate is about eleven live particles.

That arithmetic is a planning estimate, not an observed runtime count. Frame sampling, random lifetimes, and the final authored settings can change the result. A provisional capacity of 64 gives room for this small experiment, but does not establish a performance budget for a finished game.

Run the same path at half speed. Judge spacing along the ground, not just the number of particles visible in one frame. Then stop completely. If dust keeps accumulating, inspect the emission settings before adding compensating game logic.

## Export and connect the effect

Use the installed project's compatible package versions and retain the lockfile. From the folder containing `vfx-editor.prj`, run:

~~~sh
npx nixie-fx validate .
npx nixie-fx export .
~~~

Read the diagnostics even when validation succeeds: warnings do not necessarily fail the command. Copy the whole exported bundle into the host application's public assets, preserving its relative paths.

Follow the [NixieFX Three.js runtime guide](https://nixiefx.com/threejs-runtime/) to fetch the manifest and compiled effects, validate them with `loadVfxExportBundle` for `three3d`, and create a `ThreeVfxRenderer` using the host scene and camera. Select the exported dust effect by its actual ID.

Mount the effect under a stationary scene root. Update its position from the character's world-space ground-contact point; do not parent the entire renderer to the moving character.

The following is integration code for an existing render loop, not a complete application. `dust` is the created effect instance, `vfx` is its owning VFX renderer, and `footWorldPosition` is the contact position calculated by the game after movement:

~~~js
function updateDust(deltaSeconds, footWorldPosition) {
  dust.setTransform({
    position: [
      footWorldPosition.x,
      footWorldPosition.y + 0.02,
      footWorldPosition.z,
    ],
  });
  vfx.update(deltaSeconds);
}
~~~

Call this once per frame with seconds, after movement and before drawing the scene. Do not also advance the same instance separately. On scene teardown, destroy the VFX renderer.

## Test the visible failure cases

Keep the camera fixed while testing movement first. Otherwise, a camera following the marker can make stationary dust appear to move. Next, orbit the camera and inspect whether the cloud obscures the marker or intersects the floor awkwardly.

Add a teleport test separately from ordinary travel. A large position jump should not be presented as normal movement through the intervening space. Reset or recreate the dust instance at the destination as an explicit host behavior, then verify that no unintended bridge of particles appears.

For a reduced-motion option, let the host omit the decorative effect. Test that movement remains understandable without it. Do not claim that the particle editor automatically supplies the game's accessibility preference handling.

## Record limitations with the result

Editor preview bloom settings are not exported. The Three.js instanced path does not apply texture atlas subframes through `offset` and `repeat`; this recipe avoids that dependency. Neither limitation should be described as an unsupported dust-trail feature generally.

Inspect the exported `three3d` support report and runtime diagnostics for the actual effect. Resolve blockers and describe any remaining approximations. Package versions matter, so record them with the result instead of claiming identical behavior across unspecified builds.

Before presenting this as a working example, capture real evidence of moving, stopping, turning, and teleporting. Keep the effect source, exported bundle, host integration, and reproduction instructions together. No screenshots, runtime measurements, or successful test results are claimed here.

*Disclosure: Prepared with AI assistance for NixieFX marketing. Runtime verification of this recipe is pending.*
