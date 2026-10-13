---
layout: post
title: "Give campfire embers a fixed wind and a repeatable camera"
date: 2026-10-13
permalink: /2026/10/13/campfire-embers-fixed-wind/
description: "A fixed seed, timestep, and camera make directional ember motion easier to compare."
---

A campfire effect does not need every ember to fly straight upward. A small directional drift can make the air around the fire legible. The difficulty is evaluating that drift without accidentally comparing different random particles, different ages, or different camera angles.

This NixieFX tutorial builds a controlled ember scene with one emitter, one wind vector, and three named camera views. The example uses NixieFX 0.1.20 with Three.js r185. It is a scene fixture for inspecting motion, not a fire or weather simulation.

## Separate the campfire prop from the particles

The log and ground marker are ordinary Three.js geometry. They remain fixed while the ember emitter changes. This gives the viewer a reference for the spawn area and makes the horizontal drift easier to judge.

The effect origin is `[0, 0.2, 0]`. A box measuring `[0.6, 0.15, 0.6]` distributes births around the log. The emitter produces 36 particles per second, each with a two-second lifetime, and reserves capacity for 128 particles. The capacity is intentionally above the approximate 72-particle steady population.

Each ember starts with a small upward velocity and a little lateral variation. Its initial size is 0.08 world units. Additive blending produces bright overlaps, but the example includes no bloom pass and no particle lights. An orange particle is not evidence that the surrounding scene is being illuminated.

## Add wind as an authored setting

Enable the external-forces module and give it a fixed wind vector. For this example the values are `[1, 0, 0.25]`, making positive X the dominant horizontal direction with a smaller positive Z component.

```js
emitter.modules.externalForces = true;
emitter.advanced.externalForces.wind = [1, 0, 0.25];

const embers = vfx.createEffect(definition, {
  position: [0, 0.2, 0],
  seed: 2444
});
```

These values are the fixture's authored parameters. Do not label them as meters per second without defining a world-unit convention and checking the force model. The example's purpose is to inspect directional behavior, not to reproduce measured wind.

The test effect begins with the runtime's default effect factory, then explicitly changes the emission, lifetime, initialization, color, and external-force fields. That keeps the test reproducible without requiring an unstated editor preset.

## Freeze the comparison conditions

Use a fixed seed and a fixed simulation step. The baseline camera sits at `[8, 7, 9]`, looks toward `[0, 1, 0]`, and uses a 45-degree perspective field of view. Record those values next to any screenshot used for comparison.

The replay control restarts the instance with the same seed. Pause freezes the update loop. Camera controls change the view without advancing particle age. After a camera change, the fixture calls `setCamera` and a zero-time update to refresh billboard orientation while the simulation remains paused.

An overhead view helps distinguish lateral motion from upward motion. A low-angle view reveals how high the embers rise relative to the log. Keep both, because a single camera can make a diagonal path appear nearly vertical through projection alone.

## Verify repeatability before tuning the wind

A small automated check created the ember effect twice with seed 2444, advanced each run by 90 steps of 1/60 second, and compared its particle debug-transform data. The two results matched. At that point, each run reported 53 active and visible particles and one instanced draw call.

That count describes the sampled time of 1.5 seconds, before the approximate steady population is reached. It should not be presented as the peak cost of the effect. The browser fixture exposes the current and peak counts so a longer run can be inspected directly.

The Three.js support report for this definition was `supported`, with no reported blockers or warnings. Those results apply to the tested definition and package version. They do not certify every possible external-force combination or every device.

## Use the scene as a comparison checklist

Replay the effect and pause at the same age before comparing a changed wind vector. Keep the spawn box, lifetime, initial velocity, scale, and camera unchanged. Otherwise a larger drift could be caused by older particles or a longer lifetime rather than the wind setting.

Check that particles emerge from the intended area around the log. Confirm that the dominant drift is visible from the overhead view. Inspect the low-angle view for particles that obscure the focal point of the scene. Finally, return to the ordinary gameplay angle and decide whether the motion communicates the desired direction clearly.

For implementation details, see [the NixieFX Three.js runtime reference](https://nixiefx.com/threejs-runtime/). This fixture omits terrain collisions, gust fields, heat distortion, smoke, and firelight. Those are separate systems or additional effects that need their own checks.

A useful ember study makes the comparison repeatable. Once the same particles can be viewed at the same age from a known camera, changes to wind become much easier to explain and review.
