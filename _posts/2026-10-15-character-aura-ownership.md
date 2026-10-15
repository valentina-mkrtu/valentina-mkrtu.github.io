---
layout: post
title: "Give a character aura one owner across repeated activations"
description: "A cross-renderer aura implementation plan that handles duplicate status events, character replacement, and scene cleanup."
date: 2026-10-15
---

A character aura often begins as a visual recipe and becomes a lifecycle bug. The ring looks correct during one activation, but a repeated status message creates another instance. A character respawns and the old aura remains. A scene reload adds a second update callback. None of those problems requires a more elaborate particle material to solve.

This implementation plan focuses on ownership around a cross-renderer aura. It complements a basic attachment demonstration by asking what happens when the same gameplay state arrives twice. The values below are proposed authoring settings. This article does not claim a rendered result or a performance measurement for this new lifecycle wrapper.

The wrapper is application-level code, not a built-in NixieFX status-effect manager.

## Define an aura as a state, not a click

Suppose a character has a temporary shield status. The aura should exist while that status is active. Repeated notifications that the shield remains active should update the existing effect rather than create another one. When the character disappears, its aura should be removed even if a later network message still mentions the old character.

Use a stable character-instance identifier, not just a display name. A respawn may reuse the same player name while representing a new scene object. The visual owner should distinguish those lifetimes. Keep any network ordering or authoritative status logic outside the particle adapter; the aura consumes the state that the game has already resolved.

## Author a deliberately small portable recipe

Create a portable effect named `shield-aura`. Use one continuously emitting circle, local simulation space, a narrow radius around the character, and short-lived unlit billboards. Start with 24 particles per second, a 0.5-second lifetime, and a capacity of 64. These are starting values for inspection, not a mobile budget recommendation.

Keep initial velocity low so the ring reads as an attached status. Fade alpha over lifetime and avoid a bright center that obscures the character. Use a procedural billboard initially; textures can be added after attachment and cleanup are clear. Export the effect and inspect both backend reports. Do not treat a blocked report as permission to continue.

The [NixieFX particle editor for PixiJS and Three.js guide](https://nixiefx.com/pixijs-particle-effects/) provides the authoring and integration context. This recipe deliberately avoids Three.js-specific lighting, mesh-surface emission, and GPU depth assumptions. Preview bloom does not become an exported game postprocessing pipeline.

## Put adapter differences behind a small owner

The following application wrapper expects a fully initialized renderer and an effect definition loaded from a validated bundle. Its callback receives positions already expressed in the adapter's coordinate convention. It does not load assets, convert coordinates, or decide whether a shield is active.

```js
export function createAuraOwner(renderer, definition, move) {
  const active = new Map();
  return {
    sync(characterId, enabled, position) {
      let instance = active.get(characterId);
      if (!enabled) {
        if (instance) renderer.removeEffect(instance);
        active.delete(characterId);
        return;
      }
      if (!instance) {
        instance = renderer.createEffect(definition, { seed: 13 });
        active.set(characterId, instance);
      }
      move(instance, position);
    },
    clear() {
      for (const instance of active.values()) {
        renderer.removeEffect(instance);
      }
      active.clear();
    },
    get count() { return active.size; }
  };
}
```

For Three.js, provide `(instance, position) => instance.setTransform({ position })`. For PixiJS, provide `(instance, position) => instance.setPosition(position)`. The installed 0.1.20 adapter APIs differ in other lifecycle controls, so this small wrapper does not pretend to offer a shared pause or restart operation.

## Keep movement and the frame loop separate

Call `sync` when status changes and when the character position needs updating. A duplicate active update finds the existing entry. An inactive update removes it. The application should send an inactive update for the old character identifier before discarding that object, then use the new identifier after respawn.

Advance the renderer once per host frame in seconds. Do not call its update method inside `sync`: event frequency and frame frequency are different. A repeated network event should not accelerate particle simulation. Likewise, the wrapper should not create its own animation loop for every character.

For a 2D scene, document how world or gameplay coordinates become PixiJS layer coordinates. For a 3D scene, resolve the character's world position after its animation transforms have updated. Do not apply a full character translation in both a parent container and the effect instance.

## Verify the events that usually get missed

Run this sequence in each adapter: enable one character, repeat the same enabled state, move the character, disable it, enable it again, and destroy the scene. Inspect the owner count after each step. The intended sequence is one, one, one, zero, one, zero. That is an acceptance target for the wrapper, not a claimed test result from a rendered game.

Then test two characters independently. Disabling one must not clear the other. Replace a character with a new identifier and confirm the old entry is removed. On scene exit, clear the owner before destroying scene-owned renderers and providers. Re-entering should create only the new scene's update callback.

## Limits

This wrapper removes an aura immediately when disabled. A graceful fade-out needs a separately specified completion policy and adapter-specific handling. It does not implement pooling, network reconciliation, texture loading, damage logic, or automatic cleanup for identifiers the host forgets to remove. Verify the actual export, visible ring, and device performance before treating the recipe as production-ready.
