---
layout: post
title: "Cross-renderer particle budgets: test the same effect in PixiJS and Three.js"
description: "Build a portable NixieFX reward burst, verify PixiJS and Three.js cleanup, and measure particle budgets in the real host scene."
date: 2026-10-08
---

A shared effect file does not create a shared performance budget. The same visual idea can occupy different amounts of screen space, use different rendering paths, and compete with different scene workloads in PixiJS and Three.js.

When choosing a particle editor for PixiJS and Three.js, separate the authoring benefit from the runtime measurement. Reusing an effect definition can simplify iteration. It does not prove equal frame times or identical appearance in the two hosts.

This recipe defines a small reward burst and a repeatable budgeting exercise. The twelve-particle fixture below was exercised in both runtimes. The event schedules are test inputs, not recommended limits for every device.

## Verified runtime fixture

[Open the runnable lab](https://valentina-mkrtu.github.io/demos/nixie-runtime-lab.html) and choose Cross-renderer budget. The embedded export uses NixieFX 0.1.17, PixiJS 8.22.0, and Three.js 0.185.0. Both backend checks reported supported with no validation errors. In the twenty-burst check, the host created twenty instances per renderer: forty total. After completion, the remaining-instance count returned to zero. This verifies creation and cleanup in this fixture; it is not a frame-rate comparison or a device-performance claim.

Both canvases use a 960 by 320 backing size. The Three.js orthographic camera spans 960 by 320 world units; Pixi uses a downward Y projection for screen coordinates. The circle emission is rotated into the XY plane. The fixed seed makes captures repeatable, while backend appearance still needs visual inspection.

## Start with a portable visual target

Use one short burst of twelve soft billboard particles, a proposed lifetime of 0.4 seconds, and no continuous emission. Keep the first version unlit, without trails, flipbooks, or a custom material graph. Use a consistent texture and an alpha fade that ends fully transparent.

The purpose of this simple baseline is interpretability. If several features change at once, a slower result is harder to explain. Once the baseline is understood, add one visual feature at a time and repeat the same measurement.

Author the effect against the portable profile and inspect support information for both destinations. [NixieFX’s PixiJS integration guide](https://nixiefx.com/pixijs-particle-effects/) provides the starting integration path; the host still needs an appropriate renderer, asset loading, and scene lifecycle.

## Make the screen-space workload comparable

In a 2D interface, the effect may be anchored next to a reward icon. In a 3D scene, the same effect could be positioned beside an object and projected through a camera. Matching numerical scale values is not enough to make those situations comparable.

Choose a target on-screen footprint and capture both scenes at the same viewport size and device-pixel ratio. Adjust the 3D placement or scale until the footprint is comparable. Record that adjustment so someone else can reproduce it.

Keep the backgrounds representative. Large translucent particles overlapping a busy scene can have different costs from the same particle count against an empty background. Particle count is one budget dimension, not the whole workload.

## Write the event schedule before measuring

Start with one burst per second. Then try five per second, and finally a deliberate stress case with twenty simultaneous bursts. Use a fixed event sequence so each renderer faces the same requested work.

For the proposed twelve-particle burst, twenty simultaneous instances request 240 particles before any capacity constraints or other effects are considered. That number is arithmetic from the recipe, not an observed live-particle count.

Record whether the host drops events, limits simultaneous instances, or replaces an existing effect. A beautiful performance result is misleading if one implementation quietly renders fewer bursts than the other.

## Collect frame intervals with a defined window

The helper below samples animation-frame intervals after a warm-up period. It is browser-side measurement code independent of the particle runtime. Keep the tab visible and use a production build for the comparison.

```js
export function sampleFrameIntervals({ warmup = 120, count = 600 } = {}) {
  return new Promise((resolve) => {
    const values = [];
    let previous;
    let seen = 0;
    function frame(now) {
      if (previous !== undefined && seen++ >= warmup) {
        values.push(now - previous);
      }
      previous = now;
      if (values.length < count) {
        requestAnimationFrame(frame);
        return;
      }
      const sorted = [...values].sort((a, b) => a - b);
      const percentile = (p) => sorted[Math.ceil(p * sorted.length) - 1];
      resolve({ samples: values.length, medianMs: percentile(0.5),
        p95Ms: percentile(0.95), maxMs: sorted[sorted.length - 1] });
    }
    requestAnimationFrame(frame);
  });
}
```

These values measure the host frame cadence. They are not isolated CPU or GPU timings for the effect. Browser scheduling, display refresh, scene work, and thermal conditions influence them. Run an effect-disabled baseline and repeat each configuration rather than drawing a conclusion from one trace.

## Inspect cleanup as well as the busiest frame

After the final event, let every particle finish and inspect the remaining effect instances. The host should remove completed one-shots or return them to its chosen reuse strategy. Leaving inactive instances attached can make a short test look healthy while a longer session accumulates unnecessary work.

On scene exit, remove event handlers and the update callback owned by that scene. Avoid advancing the same VFX renderer twice in one frame. Include repeated entry and exit in the test sequence because lifecycle defects often appear outside the busiest visual moment.

## Record backend limits next to the numbers

Pixi does not reproduce mesh-surface emission, lit 3D shading, prepared 3D mesh particles, or real GPU-depth behavior. Preview bloom settings also do not guarantee the same post-processing in the game. Use the support reports to identify approximations, then judge the real output.

Keep the installed package version in the report. Rendering optimizations can change between versions, and a feature that alters the drawing path may matter more than a small particle-count reduction.

## Decide the budget from the actual game

Choose the final setting only after inspecting readability, frame behavior, and cleanup on the intended devices. Keep a lower-motion or static feedback path where appropriate, and consider a host-side cap for simultaneous decorative effects.

The useful outcome is a documented recipe and a budget the game can sustain under its own workload. Sharing the authoring source makes iteration easier; measuring each destination keeps the shipping decision grounded.
