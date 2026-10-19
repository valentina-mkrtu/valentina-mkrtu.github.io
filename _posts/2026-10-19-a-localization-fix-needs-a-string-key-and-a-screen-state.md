---
layout: post
title: "A localization fix needs a string key and a screen state"
description: "Prepare an inspectable localization repair with string keys and screen context."
date: 2026-10-19
---

This guide was written with AI assistance as part of Wagglet marketing. The screen and strings below are a fictional game UI example.

A localization report that says “the button text is cut off” leaves three teams with different questions. Design needs to know which layout is failing. Engineering needs to know which resource and state produce it. QA needs a repeatable way to decide whether the correction holds.

The fix becomes easier to coordinate when the handoff contains both the string identity and the exact screen context. A translation can be linguistically correct and still fail in a constrained interface. A wider button can make one screenshot look right while covering a nearby control at another viewport.

## Freeze a small example

Suppose a reward panel contains a button with the resource key `rewards.claim_all`. Its English source is “Claim all.” For layout testing, use a deliberately long pseudolocalized label such as “[!! Claim all collected rewards !!]”. This is a synthetic stress string, not a proposed translation into another language.

The report should state the viewport, text scale, panel state, and neighboring controls. For example, use a 360 by 640 CSS-pixel viewport at 100 percent browser zoom, with the reward count visible and the panel opened from the inventory screen. Those are proposed test conditions, not measurements from an existing product.

A context screenshot should include enough surrounding UI to show the constraint. A tightly cropped label can hide the fact that the proposed wider button would overlap a dismiss control.

## Separate language judgment from layout judgment

The localization owner approves the actual target-language wording. Design defines the acceptable layout behavior. Engineering implements the bounded change. QA verifies the specified states and records failures. An agent may assist the authorized engineer, but it does not replace language review or decide that an awkward shorter translation is acceptable.

For this example, design might allow the button to grow vertically to two text lines while preserving its minimum hit area and distance from nearby controls. Engineering should not silently shrink the font to make every string fit. That can conceal the visible symptom while making the interface harder to read.

The acceptance criteria should describe the result rather than a subjective preference. “Looks balanced” is hard to test. “The full label remains visible, no glyph is clipped, and the button does not overlap the dismiss control” gives QA something concrete to inspect.

## Prepare a handoff the next person can run

```text
Resource: rewards.claim_all
Source string: Claim all
Stress fixture: [!! Claim all collected rewards !!]
Screen: inventory -> rewards panel, rewards available
Viewport: 360 x 640 CSS pixels; repeat at 390 x 844
Text scale: default and the product's supported enlarged setting
Allowed change: reward-button wrapping and container sizing
Preserve: action handler, reward calculation, resource key
Evidence: before/after screen captures, diff, and QA matrix
Escalate: any need to change approved translations or reward behavior
```

The author supplies this context before execution. The runner uses their own authorized workspace and tools to reproduce the issue and work with the coding agent. The reviewer checks the implementation and the human visual observations before accepting the result.

## Test transitions, not only the final frame

The panel may be correct when opened and fail after the reward count changes. Test the transition from several rewards to none, the disabled state during a request, and reopening the panel after completion. These cases can change the button label or available width even when the same resource key is involved.

Check the supported enlarged text setting independently of pseudolocalization. Longer words and larger glyphs stress different constraints. Also inspect keyboard focus if the product supports it; a visible label should not come at the cost of a clipped focus indicator.

For actual languages, ask the localization owner to review the final wording in context. Automated string expansion can uncover layout pressure, but it cannot establish that a translation is natural or culturally appropriate.

## Record the responsibility map

Human AI collaboration in game development works better when the handoff preserves each discipline's decision. Design approves the layout rule, localization approves language, engineering owns implementation, and QA supplies observed behavior. The acceptance owner brings those checks together rather than asking any one participant to certify everything.

[Wagglet's mobile-game studio workflow article](https://wagglet.com/blog/ai-native-mobile-game-studio-stack) provides context for coordinating these different kinds of work. The practical point here is narrower: put the string key, screen state, and acceptance evidence on the same task so a correction does not lose its reason during the handoff.

## Close the specific defect

The delivery should identify the changed component and resources, show the tested states, and list the language or device combinations not checked. If an actual translation still fails, keep that result visible. Do not report universal localization support from one expanded English fixture.

A good final review can answer three questions: which defect was corrected, which conditions demonstrate the correction, and which specialist approved the parts that required their judgment? When those answers are explicit, a small localization fix can remain small without becoming ambiguous.
