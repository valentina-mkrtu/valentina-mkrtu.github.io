---
layout: post
title: "Build a Three.js explosion around a clear origin and a finite lifetime"
description: "A bounded Three.js effect integration recipe with explicit limits."
date: 2026-10-19
---

This article provides an implementation recipe and acceptance checks; it does not report a completed explosion demo or measured performance.

An explosion effect has two independent jobs. It needs to communicate where the event happened, and it needs to leave the scene cleanly after that event. A visually elaborate burst can fail either job if its origin moves with the camera or its looping emitters never stop.

Start with a deliberately small asset: a short radial burst at a fixed world-space position. Add smoke, light, or debris only after the origin, timing, and cleanup are easy to inspect. This keeps the first integration understandable and makes later visual changes easier to evaluate.

## Define the effect before connecting the event

Use a distinct effect ID such as `world-explosion`. A proposed first pass has one radial emitter, twenty-four particles, no continuous emission, and a maximum particle lifetime of 0.6 seconds. These are art-direction starting values, not a statement about the capacity of a target device.

Author for the Three backend. Keep the initial version assetless if a procedural billboard gives the desired shape; otherwise include every referenced texture in the exported bundle. Validate the export and inspect its backend support report before calling it compatible.

The [NixieFX Three.js runtime reference](https://nixiefx.com/threejs-runtime/) is the integration reference for this three js particle editor workflow. The relevant boundary is between the exported effect and the host scene: the effect describes the burst, while the game supplies the event position and decides when it occurs.

## Keep the event position in world space

An impact event should carry a position that belongs to the scene's coordinate system. If the event originates from a child object, update its world transform and obtain the world position before passing it to the effect. Copying the child's local coordinates is only correct when the parent transform is the identity.

For a moving target, decide whether the burst should remain at the impact location or travel with the target. The first version of this recipe stays at the impact location. Capture that position once rather than updating the effect origin every frame.

This choice matters most when the target moves immediately after the hit. A stationary explosion communicates the historical impact point; a following effect communicates an ongoing attachment. Neither is universally correct, but the implementation should match the intended event.

## Use one owner for updates and cleanup

The following fragment assumes `vfx` is a configured Three renderer adapter and `effect` is the verified exported definition. It is an integration fragment, not a complete scene loader.

```js
const live = new Set();
function explode(position) {
  const instance = vfx.createEffect(effect, {
    position: [position.x, position.y, position.z],
    seed: 237
  });
  live.add(instance);
}
function updateExplosions(deltaSeconds) {
  vfx.update(deltaSeconds);
  for (const instance of live) {
    if (!instance.isActive) {
      vfx.removeEffect(instance, true);
      live.delete(instance);
    }
  }
}
```

Call the update function once from the host frame loop with seconds. Do not also advance each owned instance independently. If the scene already updates the same adapter elsewhere, merge the cleanup loop into that location instead of adding a second update.

On scene teardown, destroy the adapter and release any provider-owned resources according to the host's ownership rules. An effect ending visually is not a substitute for releasing the resources that the scene owns.

## Build a repeatable inspection scene

Use a fixed camera and a visible origin marker. Trigger one burst at the marker, wait for it to finish, and trigger another at the same position with the same seed. Compare the location and envelope before adding camera motion or multiple impacts.

Next, trigger the event on an object under a translated parent. This is a useful way to catch a local/world coordinate mistake. Move the object after the trigger and verify that the explosion stays at the intended event position.

Finally, exercise repeated events. Watch whether finished instances are removed and whether leaving the scene stops all visible activity. Record actual measurements if you profile; do not infer a frame-time budget from the small particle count.

## Treat presentation limits as part of the result

An additive burst may look clear against a dark backdrop and disappear against a bright surface. A translucent particle may intersect geometry in an undesirable way. Inspect those cases in the real camera and lighting setup rather than assuming the editor preview predicts them.

Backend support also depends on the exported settings and installed runtime version. Treat blocked settings as unsupported and document partial-support warnings. Do not promise that a Pixi rendering of the same asset will have the same depth or lighting behavior as the Three scene.

## Know when the first version is done

The first version is ready for review when the event appears at the correct origin, ends within its authored envelope, leaves no retained finished instances, and remains understandable in the intended scene. Attach the actual export, host source, and a runtime capture to that review.

Only then add secondary layers. A smoke tail, shock ring, or debris emitter should earn its place by improving the event's readability. Keeping the initial burst finite and well-positioned gives those additions a stable foundation.
