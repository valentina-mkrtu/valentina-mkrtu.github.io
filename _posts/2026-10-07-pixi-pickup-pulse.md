---
layout: post
title: "Make a PixiJS pickup pulse confirm one event"
description: "A bounded pickup effect with event deduplication, scene coordinates, and a clear cleanup path."
date: 2026-10-07 09:00:00 +0400
published: true
---

A pickup pulse should answer a small question: did that item get collected? If the animation fires twice, follows the wrong object, or continues after the scene closes, it makes that answer less clear.

The useful design starts with the event that owns the effect. A pointer press is an intention. A confirmed collection is an outcome. For an online game, those may happen at different times. The pulse described here belongs to the confirmed collection event, while immediate button feedback can remain a separate interaction.

This example uses a small, non-looping particle burst in PixiJS 8.19.0 with NixieFX 0.1.17. Try the [pickup inspection fixture](/demos/pickup-pulse.html): confirm a pickup, replay the same event, move the parent container, and enable reduced motion. In the browser check, replay left the inventory and pulse totals unchanged; moving the parent changed the VFX position from 310,175 to 440,175; reduced motion preserved the inventory update without another pulse; disposal left zero live effects. These are fixture observations, not production-performance measurements.

## Give the pulse a short visual contract

Start with twelve small, unlit billboard particles spreading outward in the two-dimensional game plane. Use a short lifetime, around a third of a second, with opacity fading to zero. Keep the overall effect timeline long enough for the final particle to disappear, and disable looping on both the effect and emitter.

Avoid a long fountain for this interaction. A pickup may occur close to health text, an objective marker, or another item. The effect should draw attention to the collection point without obscuring the next decision.

When selecting a [pixi js particle editor](https://nixiefx.com/pixijs-particle-effects/), check the runtime path as well as the preview. With NixieFX, the game loads an exported bundle. Keep the editable project separate from the files served to the game, and review the Pixi backend support report before integrating the effect.

## Connect the event to the VFX layer’s coordinates

Suppose the pickup sprite lives inside a translated world container while the VFX layer sits elsewhere in the scene graph. Copying the sprite’s local x and y directly into that layer can put the pulse in the wrong place.

Convert the pickup location into the VFX layer’s coordinate system before spawning. Capture it before removing the sprite. The code below is an integration fragment: it assumes that the application, exported bundle, renderer, and dedicated VFX container have already been initialized.

```js
// Existing objects: app, bundle, vfx, vfxLayer.
// vfx is a PixiVfxRenderer parented to vfxLayer.
const definition = bundle.effectsById.get("pickup-pulse");
if (!definition) throw new Error("Missing pickup-pulse export");

const seen = new Set();
const live = new Set();
const motion = matchMedia("(prefers-reduced-motion: reduce)");

function onPickupConfirmed(eventId, pickupSprite) {
  if (seen.has(eventId)) return;
  seen.add(eventId);
  // The authoritative game state updates the inventory separately.
  if (motion.matches) return;
  const point = vfxLayer.toLocal(pickupSprite.getGlobalPosition());
  live.add(vfx.createEffect(definition, {
    position: [point.x, point.y, 0],
    seed: 71
  }));
}

function tick(ticker) {
  vfx.update(ticker.deltaMS / 1000);
  for (const instance of live) {
    if (!instance.isActive) {
      vfx.removeEffect(instance, true);
      live.delete(instance);
    }
  }
}
app.ticker.add(tick);

function disposePickupScene() {
  app.ticker.remove(tick);
  live.clear();
  seen.clear();
  vfx.destroy(); // this scene owns this renderer
}
```

Use a genuine event identifier from the collection flow. A sprite name alone is not enough if the game reuses that name for several items. The example keeps identifiers for one bounded scene; a persistent world needs a bounded deduplication strategy based on its event protocol.

## Let the game own the reward

The visual callback must not add currency, unlock an item, or decide whether a collection succeeded. It reacts to state that has already been accepted by the game. This keeps replayed visual events from granting repeated rewards.

For reduced motion, the example skips the burst. Keep the ordinary inventory update and a readable confirmation available. A player should receive the same information and reward when particles are disabled.

The fixed seed helps compare repeated runs of the effect during review. It does not make an entire game session deterministic, and it does not remove differences caused by camera position, renderer settings, or the surrounding scene.

## Test the event boundaries

Send the same confirmation twice and verify that only one pulse appears. Send two different collection events at different locations and verify that each uses the correct point. Move the parent container and repeat the check to expose coordinate mistakes.

Next, leave the scene immediately after a pickup. Confirm that the ticker callback and renderer are released. Re-enter the scene and collect another item. A duplicate animation loop can remain invisible in a screenshot while doubling simulation work over time.

Also test with particles disabled through the reduced-motion preference. The reward state and readable feedback should remain correct. Record the actual browser, viewport, and build when capturing the result.

## Keep the renderer limits explicit

This recipe uses planar, unlit billboards. It does not depend on prepared three-dimensional mesh particles, mesh-surface emission, scene lighting, or GPU depth behavior from the Three.js adapter. Those features should not be assumed to transfer to PixiJS; lit materials render unlit there.

Check the exported support warnings for the specific effect and installed runtime version. A supported export is a compatibility check, not a guarantee that the effect meets your frame budget. Measure it with the rest of the game running, and keep the pickup confirmation useful even when the decorative pulse is absent.
