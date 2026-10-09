---
layout: post
title: "Play a quest-completion effect once, even when the event arrives twice"
description: "A PixiJS quest-marker design with explicit timing, duplicate-event handling, and reduced-motion behavior."
date: 2026-10-09
---

A quest-completion animation should celebrate a change in game state. If it plays twice because the client receives a duplicate notification, the decoration starts telling a different story from the quest system. If it replays every time the panel opens, a completed objective can look newly completed forever.

This walkthrough proposes a small marker effect with a clear contract: show completion immediately, optionally play one brief animation, and preserve the completed state after the animation ends. The implementation and capture still need to be verified in a real PixiJS project; the timings below are design targets, not observations from a finished demo.

## Define the event you are celebrating

Choose a stable completion identifier from the game’s authoritative state. A quest ID alone may be insufficient for repeatable quests, so include the completion revision or instance identifier. Do not invent a new random identifier each time the UI receives the same message.

The host game decides whether a completion is new. The effect only visualizes that decision. Reward grants, inventory changes, and persistence must not depend on whether a particle animation successfully loads or finishes.

For a worked example, imagine a delivery quest with completion key `delivery-quest:run-42`. The first confirmed event changes the marker to a check and starts the celebration. A second event with the same key updates the text if necessary but does not restart the burst.

## Set a modest timing contract

Use a total visual window of roughly 0.6 seconds as an initial design target. The completion check appears immediately. A few particles expand outside the marker, then fade before the player needs to read the next objective. Keep the effect away from the quest title and reward amount.

This target needs adjustment against the real interface. A large tablet panel and a narrow phone panel may have different available space. Test the smallest supported layout first, because text overlap is harder to notice in a roomy editor preview.

In reduced-motion mode, skip the moving particles and retain the static check plus “Completed” label. Completion should remain understandable without movement, color, or sound alone. A player who closes the panel during the celebration should still find the completed state when they return.

## Put duplicate handling in the host controller

The following pseudocode describes application logic. Its callbacks are integration boundaries, not names of NixieFX methods.

```js
const celebrated = new Set();

function onConfirmedCompletion(event, ui, effects, reducedMotion) {
  ui.showCompleted(event.questId);
  if (celebrated.has(event.completionKey)) return;
  celebrated.add(event.completionKey);
  if (!reducedMotion && ui.isVisible(event.questId)) {
    effects.playQuestBurst(event.questId);
  }
}
```

This in-memory set handles duplicates only for the lifetime of the controller. It is not durable deduplication across reloads. Decide whether the game should persist “already celebrated” state or derive it from an existing acknowledgement. A repeating quest needs a new completion key for each legitimate run.

Also define what happens when the event arrives while the panel is closed. For this example, the completed marker is enough; reopening does not replay missed celebrations. A different product may queue them, but that is a separate interaction decision and should be tested deliberately.

## Integrate one effect without making it game logic

Use the [visual particle editor for PixiJS](https://nixiefx.com/pixijs-particle-effects/) to prepare the decorative burst. Load the exported bundle through the documented Pixi integration and place the effect in a dedicated visual layer. Keep the persistent check and label as ordinary UI elements.

Choose a flat, unlit asset for this example. Pixi does not support mesh-surface emission or lit shading in the documented backend. Review the actual export report as well, since a particular asset can introduce warnings beyond these general limits.

The renderer should advance once per frame. When a one-shot completes, remove the finished instance; when the panel is destroyed, release the resources it owns. Do not use a completion callback from the effect as permission to grant the quest reward.

## Test state transitions, not just the pretty frame

Start from an incomplete quest and confirm one completion. Record the state before the event, during the animation, and after it settles. Then send the same completion event twice and verify that there is still only one celebration.

Close the panel during playback. Reopen it and confirm that the check and label persist without replay. Repeat with reduced motion enabled before completion, then with the preference changed while the panel is open. The application should stop decorative motion promptly and retain the meaningful state.

Finally, complete a new instance of the repeatable quest. Its distinct key should permit one new animation. This catches implementations that suppress all future celebrations after the first quest ID has been seen.

## Publish the evidence with the example

Attach the authored effect, exported bundle, source revision, and exact dependency versions to the eventual demo. Capture the duplicate-event case as well as the normal case. List any unsupported settings and any untested devices beside the result.

Until those artifacts exist, this is a reproducible design and integration plan rather than a verified runtime result. The acceptance criterion is narrow: one confirmed completion produces one optional celebration, and every path leaves the player with a readable, correct quest state.
