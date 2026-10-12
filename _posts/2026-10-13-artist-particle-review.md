---
layout: post
title: "Review a particle-polish change with a contact sheet and an artist's decision"
description: "A practical review packet for game effects: a bounded brief, comparable frames, and approval tied to the delivered revision."
date: 2026-10-12
---

The hardest part of reviewing a particle-polish change is often knowing what to compare. A developer sends a new clip. The artist remembers an earlier version. Someone says the effect feels stronger, but nobody is looking at the same frame, camera, or background.

A contact sheet can turn that conversation into a reviewable artifact. Put the reference and the candidate at matching moments, preserve the full gameplay frame, and ask the artist to record a decision against a specific revision.

This article describes an illustrative process for a sword-hit effect. No runtime capture or artist approval is claimed. The proposed artifact is deliberately small enough to include in an ordinary development task.

## Start with a visual question

Suppose a hit spark covers the enemy's attack indicator. The effect looks energetic in isolation, but at the intended camera distance it competes with a cue the player needs to react to.

The task should ask whether the attack indicator remains readable while the impact still feels connected to the weapon strike. That question gives the artist something concrete to judge. “Make the particles more polished” does not.

Use a game scene with the actual indicator, enemy, weapon, and background. Preserve the framing for every candidate. A close-up may help diagnose a texture edge, but it should not replace the normal gameplay view that exposes the readability problem.

## Prepare a brief the implementer can execute

The engineer supplies the reproducible scene and identifies the relevant effect asset. The artist supplies the visual priority: preserve the attack cue, keep the impact localized, and avoid changing the weapon's perceived weight. A developer or agent operator performs the edit using their own authorized tools and repository access.

The brief for this example can read:

~~~text
Objective: Keep the enemy attack indicator readable during a sword hit.
Allowed edits: Spark spread, opacity, size, lifetime, and color progression.
Preserve: Hit timing, damage behavior, indicator animation, camera framing.
Reference: Actual gameplay capture plus artist-annotated target frames.
Required candidates: One scoped revision, with settings recorded.
Technical checks: One trigger per hit; normal cleanup after scene exit.
Visual acceptance owner: Artist assigned to combat readability.
Integration acceptance owner: Engineer assigned to the combat scene.
~~~

Do not let the executor silently solve the problem by delaying the attack indicator. That changes the gameplay cue rather than polishing the effect. If the requested visual improvement is impossible inside the allowed scope, return the conflict to the people who wrote the brief.

## Construct the contact sheet

Use three columns: the current effect, the artist's target reference, and the candidate runtime result. Use four rows sampled at the hit, the early expansion, the largest overlap, and the fade. Choose those moments from the actual animation and label their elapsed times.

The target reference may be an annotated frame or a drawing. Label it clearly so nobody mistakes it for a captured implementation. Circle the attack indicator and mark the area in which bright sparks may appear. Keep annotations outside the important visual regions where possible.

For a completed sheet, use descriptive alt text such as: “Current sword-hit effect, annotated visual target, and candidate effect compared at four matched times, with the enemy attack indicator marked.” The screenshot and candidate columns must come from your game. This illustrative process does not include a captured implementation.

Save the full-speed clips alongside the sheet. Still frames show overlap, while motion reveals whether the effect feels late, disconnected, or unexpectedly persistent. Record the scene revision, effect revision, viewport, camera settings, and random seed where the implementation supports one.

## Make disagreement actionable

Imagine that the candidate clears the indicator during the expansion but hides it during the fade. “Almost there” leaves the next edit ambiguous. The artist can instead point to the fourth row: the remaining bright particles cross the indicator at the moment the enemy becomes dangerous.

The next attempt can then adjust the tail without rebuilding the whole burst. Preserve the original reference and add the new candidate. Do not replace every image in the packet and force the reviewer to reconstruct what changed.

That continuity is valuable in a game studio AI workflow because implementation may move between people and agents. The visual decision should survive that handoff. [Wagglet's mobile game studio stack article](https://wagglet.com/blog/ai-native-mobile-game-studio-stack) provides further background on the broader coordination context.

## Record the artist's actual sign-off

The artist's decision should name the reviewed revision, the reference used, the outcome, and any remaining exception. Leave the fields pending until the person has reviewed the evidence.

~~~text
Artist: PENDING
Reviewed revision: PENDING
Reference version: PENDING
Decision: PENDING — accept / rework
Reason and frame references: PENDING
Known visual exceptions: PENDING
Engineer integration decision: PENDING
~~~

An artist's acceptance covers the visual question. It does not establish performance on a device that was never tested. Likewise, an engineer's successful integration check does not substitute for the artist's judgment about composition.

## State what the review leaves open

The packet covers a particular scene, camera, and set of review conditions. Different enemy sizes, color-vision needs, busy backgrounds, and simultaneous hits may require additional checks. List those cases explicitly instead of stretching one approval to cover every context.

A useful final delivery contains the effect change, the evidence, and the decision together. Anyone opening the task later should be able to identify the visual problem, inspect the correction, and see who accepted that specific result.
