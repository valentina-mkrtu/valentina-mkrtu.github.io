---
layout: post
title: "Build a readable Three.js explosion with a short burst and a quiet tail"
description: "An original explosion recipe with separate visual phases, runtime integration instructions, and explicit verification limits."
date: 2026-10-12
---

An explosion has a small amount of time to communicate several things: where the impact happened, how large it was, and when the player can read the scene again. Adding particles can make all three harder to understand.

This recipe separates those jobs into a short center flash, an outward burst, and a fading tail. It is a proposed starting configuration for a Three.js game, not a measured preset or a completed runtime demonstration. The counts and timings below are design choices to test in your scene.

## Choose the scene before choosing the effect

Use a small test arena with the actual gameplay camera, one target object, and a floor. Keep the target near a wall or another occluder so you can inspect depth behavior. A particle preview on a black background will not answer whether the explosion hides nearby enemies.

Set a consistent scene scale. For this exercise, treat the target as roughly one world unit wide and place the explosion at its center. If your game uses a different scale, change the effect size and velocity together. Do not compensate for a distant camera by increasing particle count first.

If you are evaluating a [three js particle editor](https://nixiefx.com/threejs-runtime/), NixieFX's Three.js runtime reference describes the integration boundary used here: author the effect, export a runtime bundle, and load it through the renderer adapter. Keep the design recipe separate from that loading code so each can be reviewed independently.

## Build three phases with different jobs

Start a Three.js-targeted effect named `arena-explosion`. Use simple unlit billboards initially. Avoid custom materials and external textures until the timing reads well; each extra asset introduces another variable to diagnose.

For the center flash, try one particle at the origin with a lifetime of about 0.08 seconds. Make its initial size roughly half the target's width and bring its alpha down quickly. The purpose is to mark the impact location, not wash out the frame. If the scene already has strong highlights, a pale orange may separate better than pure white.

For the main burst, try 24 particles emitted once at the start. Give them outward motion, lifetimes between 0.25 and 0.45 seconds, and sizes small enough that the target silhouette remains legible. Use a warm-to-darker color progression and shrink the particles as they expire. Spread should communicate the radius without implying a gameplay damage area you have not actually implemented.

For the tail, start with six slower particles delayed by about 0.05 seconds. Let them fade over roughly 0.6 seconds. Their job is to connect the impact to the aftermath. Keep their opacity low so they do not become a second, larger explosion.

These are proposed values. If the effect reads poorly, change one phase at a time and compare at the same camera position. Record each candidate's settings instead of relying on memory.

## Export before judging the runtime result

Export the project and inspect the Three.js support report. Resolve blocked settings before loading. If any feature is marked partial, write down the approximation and decide whether it is acceptable for this effect.

Serve the generated bundle from the game's static assets. Install the `nixie-fx` runtime alongside the game's compatible Three.js dependency and record the actual installed versions. Fetch the manifest and its listed effect files, then validate them with the documented bundle loader for the `three3d` backend.

Create a Three.js VFX renderer using the existing scene and camera. Look up `arena-explosion` by its actual exported identifier; do not assume its display name and identifier match. Create the effect at the target position, choose a fixed seed for comparisons, and advance the VFX renderer exactly once from the host frame loop using elapsed seconds.

This is an integration sequence, not a complete application listing. It assumes that your host already supplies the camera, scene, render loop, resize handling, and asset server.

## Review the burst in motion

Use this capture checklist for the first runtime pass:

~~~text
Record: runtime versions, build revision, seed, camera, viewport
0.00 s: impact center agrees with the target position
0.08 s: flash recedes without hiding nearby gameplay cues
0.25 s: outward motion reads clearly at normal playback speed
0.45 s: main burst is ending; target area becomes readable
0.70 s: tail has cleared; no unexpected loop remains
Repeat: several triggers, then scene teardown and re-entry
~~~

Treat these times as observation points, not guaranteed outcomes. Capture the actual result and note any variance from the intended timing. Include a wall-crossing view to inspect occlusion and a normal gameplay view to inspect readability. A slow-motion recording is useful for diagnosis, but it should accompany normal-speed playback.

## Keep the limitations visible

Editor preview bloom is not a promise that the deployed scene will have the same glow. Configure and review the game's own postprocessing separately. A bright particle color also does not prove that nearby geometry is illuminated.

This recipe excludes mesh-surface emission, lit mesh particles, and custom material graphs. It does not claim cross-renderer parity, a particular frame rate, or collision with the level. Depth and transparent overlap still need inspection in the actual Three.js scene.

The finished evidence should include the export report and a runtime capture. Until those exist, this remains a reproducible authoring plan. Its success criterion is simple: the impact reads immediately, and the player can read the scene again before the next important event.
