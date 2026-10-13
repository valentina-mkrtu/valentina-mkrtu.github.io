---
layout: post
title: "Find the mobile effect budget before choosing a particle cap"
description: "A repeatable frame-pacing benchmark for PixiJS and Three.js effects, with explicit device, throttling, and sample limits."
date: 2026-10-13 12:00:00 +0400
published: true
---

A particle cap becomes useful only when it belongs to a specific scene, device, and quality target. Two hundred tiny sparks over an empty background and two hundred large translucent smoke sprites can ask very different things of a renderer. A desktop preview cannot settle that difference for a phone.

This article proposes a benchmark protocol for cross-renderer web-game teams. It includes a small collector and an illustrative decision, not measured NixieFX performance results. AI assisted with the writing. No physical-device benchmark was performed for this post.

## Freeze the scene before timing it

Create a dedicated route in the game containing the normal background, camera, UI, and gameplay workload. Add a switch between zero, one, two, four, and eight simultaneous copies of the same effect. Preload assets before starting; loading performance deserves a separate experiment.

For a shared test effect, use a point emitter, a constant rate of 40 particles per second, a one-second lifetime, a 64-particle cap, and small unlit billboards. Disable trails and child effects for this first pass. These are proposed fixture settings, not a recommended production budget. Keep the authored bundle, texture, camera, projected sprite size, and seed schedule fixed. Give each copy its own predetermined seed and position; reuse that list for every trial.

A [particle editor for PixiJS and Three.js](https://nixiefx.com/vfx-runtime-docs/) can supply a portable export, but inspect its support reports before comparing results. NixieFX shares deterministic simulation across its adapters; rendering costs still differ. Mesh-surface emission and lit shading are unsupported on PixiJS, which also lacks Three.js GPU depth testing. Preview bloom is not exported. Exclude those features from this fixture, and record any approximations instead of silently treating the backends as equivalent.

## Write the device contract

Create one result row per physical device, browser, renderer, and quality setting. Record the device model, OS version, browser version, package lockfile revision, effect-bundle hash, viewport size, actual drawing-buffer size, effective pixel ratio, and refresh rate. Note battery level, charging state, power-saving mode, and whether the phone was already warm.

On a desktop, run separate unthrottled and CPU-throttled trials. Record the exact DevTools preset or slowdown factor and host hardware. Chrome's Performance documentation describes CPU throttling as relative to the host computer; it does not reproduce a phone's processor architecture. A narrow viewport and slower CPU also do not reproduce a mobile GPU or its thermal behavior.

Use desktop throttling to expose regressions. Make release decisions on named physical phones. Keep each device's measurements separate, including Android and iOS browsers where both are supported. A result from one phone is evidence about that configuration, not a statement about every mobile user.

## Collect a bounded sample

For each load level, warm the scene for ten seconds, then collect sixty seconds of animation-frame intervals. Repeat three times, with a fresh scene and identical seed sequence. Alternate ascending and descending load order across trials, and give physical devices a consistent cooldown between them. Also run a separate sustained-load session if overheating is a concern; a short trial cannot establish thermal stability.

The following browser helper measures callback spacing. Call it once after warm-up while the existing game loop continues normally; do not advance the effect renderer again inside this collector. Its timestamps are milliseconds, independently of the runtime update API's seconds convention.

```js
async function sampleFramePacing(seconds = 60, budgetMs = 20) {
  const intervals = [];
  let first, previous, invalid = document.hidden;
  const onVisibility = () => { if (document.hidden) invalid = true; };
  document.addEventListener("visibilitychange", onVisibility);
  try {
    await new Promise(resolve => {
      function frame(now) {
        if (first === undefined) first = now;
        if (previous !== undefined) intervals.push(now - previous);
        previous = now;
        if (invalid || now - first >= seconds * 1000) resolve();
        else requestAnimationFrame(frame);
      }
      requestAnimationFrame(frame);
    });
    if (invalid) throw new Error("Discard trial: page was hidden");
    const sorted = [...intervals].sort((a, b) => a - b);
    const percentile = p => sorted[Math.ceil(sorted.length * p) - 1];
    return {
      samples: intervals.length,
      p50Ms: percentile(0.50),
      p95Ms: percentile(0.95),
      overBudgetPercent:
        100 * intervals.filter(ms => ms > budgetMs).length / intervals.length,
      intervals
    };
  } finally {
    document.removeEventListener("visibilitychange", onVisibility);
  }
}
```

Integrate this helper into the test route and retain its returned raw samples with the run metadata. It does not construct a NixieFX scene, measure GPU time, or certify presentation smoothness. Callback timing includes scheduling effects and may miss compositor problems. Use a performance trace to investigate failures. Avoid logging every frame, and use the same instrumentation in baseline and loaded trials.

## Turn measurements into a decision

Choose an acceptance rule before inspecting results. For a proposed 60 Hz target, a team might require p95 callback spacing below 20 ms and fewer than 5% of intervals above 20 ms in every repeat. This allows some tolerance around the nominal 16.7 ms interval; it is an example policy, not a guarantee of smoothness.

Consider a hypothetical device where baseline p95 is 17 ms, two copies produce 19 ms, and four copies produce 26 ms. If the two-copy trials also pass the over-budget rule, two is the largest tested passing load. Test three copies next to narrow the boundary. Do not call the 9 ms difference between baseline and four-copy p95 an isolated GPU cost: percentiles from separate runs do not subtract into component timings.

Then repeat the candidate cap in the busiest gameplay scene. Reserve room for overlapping effects, larger screen coverage, and scene transitions. A cap that passes an isolated fixture can still fail during combat.

## Publish the limits alongside the cap

Keep all three trial summaries, sample counts, raw intervals, and any discarded-run reasons. Consecutive frames are correlated, so thousands of intervals are not thousands of independent experiments. Three repeats offer a small diagnostic sample, not population-wide confidence.

Before accepting a budget, verify that:

- Both renderer reports were checked and any approximation was disclosed.
- Device and drawing-buffer settings accompany every result.
- The baseline and candidate cap passed the same predefined rule.
- Physical-phone trials support any claim about mobile performance.
- The real gameplay scene and a sustained session have been checked separately.

Until those records exist, describe the cap as provisional. A reproducible procedure is valuable on its own; inventing numbers to make it look finished would make the eventual performance decision worse.
