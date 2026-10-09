---
layout: post
title: "Keep a beam-endpoint effect attached to the hit, not to a stale position"
description: "A Three.js integration plan for a target-following endpoint accent with explicit start, stop, and cleanup behavior."
date: 2026-10-09
---

A beam can point at the correct target while its endpoint particles linger somewhere else. The common cause is a split update: the beam uses the latest hit point, but the effect still uses the position captured when firing began. Another failure appears when firing stops and the endpoint continues to animate.

Treat the endpoint as a small stateful part of the weapon presentation. It should read the same hit result as the beam, have a defined response to target loss, and stop according to an explicit visual rule. This article is an implementation plan; the asset and runtime behavior still require a real project run.

## Separate the beam from its endpoint accent

The host game owns aiming, collision queries, damage, and beam geometry. The particle effect decorates a valid hit point. It should not decide whether a target was hit or how much damage occurred.

For a worked example, imagine a training scene with a moving target and a beam that can be held on or released. The endpoint accent runs only while the beam is firing and the host reports a valid hit. When the ray misses, the accent stops instead of following an arbitrary last-known target.

Choose whether the design uses a repeating accent during contact or a single impact burst when contact begins. Those are different behaviors. This example uses a repeating contact accent, with a short one-shot impact left as a possible separate asset rather than an implied built-in feature.

## Give one update owner the latest hit point

Resolve the current hit before synchronizing the effect. Both the beam mesh and the endpoint controller should consume that result from the same frame. Avoid separate ray queries with slightly different timing or coordinate transforms unless the distinction is intentional.

Keep the endpoint position in the coordinate space expected by the integration. Test a target under a translated or rotated parent so a world/local mismatch becomes visible. An effect that looks correct only at the scene origin has not passed the positioning test.

The controller below is pseudocode. The adapter callbacks are host functions to implement against the installed runtime, not library API names.

```js
let contactActive = false;

function syncEndpoint(firing, hit, adapter) {
  const validContact = firing && hit !== null;
  if (!validContact) {
    if (contactActive) adapter.stopContact();
    contactActive = false;
    return;
  }

  adapter.setContactPosition(hit.worldPosition);
  if (!contactActive) adapter.startContact();
  contactActive = true;
}
```

Keep the simulation update outside this state transition function and call it once per host frame. Starting a fresh instance on every frame is unnecessary for this design and makes cleanup harder to inspect.

## Choose what stopping means

An immediate stop is useful when a stale accent would falsely suggest continued contact. A graceful finish can work when the remaining particles are clearly a fading remnant. Pick the rule based on the game’s visual language and test it during rapid firing changes.

The [NixieFX Three.js runtime guide](https://nixiefx.com/threejs-runtime/) distinguishes immediate stopping from allowing existing particles to finish. It also documents transform updates and restarting. Use those verified controls when implementing the host adapter, and confirm their behavior in the installed package version.

If the effect uses world-space particles, moving the emission origin should not be confused with moving every existing particle to the new target. Decide whether the desired look is a trail of fading contact dust or a tightly attached energy accent. Then verify the authored simulation space and the target-switch result together.

## Check target changes as carefully as movement

A slowly moving target is the easy case. Switch between two targets far apart and inspect whether the old endpoint clears or finishes according to the chosen rule. Rapidly alternate valid hits and misses. Release the trigger while the target is moving, then fire again without reloading the scene.

Record the expected behavior before running these cases. For the immediate-stop version, no endpoint should remain at the previous target after contact is lost. For the graceful version, any remaining particles must fade within the authored lifetime and must not continue emitting.

Also test pause, backgrounding, and scene exit. The endpoint should not produce catch-up bursts after a long pause or retain listeners when the training scene is destroyed. These are host lifecycle requirements, not properties guaranteed by an attractive particle asset.

## Avoid promising unsupported visual behavior

This plan does not supply collision detection, surface-normal alignment, damage feedback, or beam geometry. Each belongs to the host project and needs implementation if the final demonstration depends on it. The runtime reference documents `clockSpace` as reserved and unused; do not rely on it to solve coordinate-space behavior.

Review the actual export’s Three.js support diagnostics and publish any warnings or blockers. A working endpoint asset does not establish support for every editor setting, and an editor image does not prove the same appearance in the final scene.

## Make the final demonstration inspectable

Include the authored asset, export, source revision, dependency versions, and a capture of the moving-target and target-loss cases. State which stop mode the demo uses. A short recording should show the target, the beam, and the endpoint together so their timing can be compared.

Until those artifacts exist, describe this as a build plan rather than a tested three js particle editor example. The finished result should demonstrate one clear contract: the accent appears at the current valid hit, reacts predictably when contact changes, and releases its work when the scene ends.
