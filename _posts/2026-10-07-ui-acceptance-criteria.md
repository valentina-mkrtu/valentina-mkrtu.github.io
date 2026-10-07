---
layout: post
title: "Write UI acceptance criteria that survive the implementation"
description: "A practical task template for visual references, interaction checks, and a revision-specific review verdict."
date: 2026-10-07 09:00:00 +0400
published: true
---

A request to “make the dialog match the design” leaves too many decisions to the person doing the implementation. Which design revision? At which screen size? Does a close button need to restore focus? What happens when the title is twice as long as the example?

Those details become especially important when an agent can produce a plausible interface before the team has agreed on what success means. Fast implementation does not remove the need for an acceptance contract. It makes an unclear contract easier to overlook.

This worked example uses a fictional notification-preferences dialog. The goal is modest: replace the current layout while preserving how preferences are saved. The template below is a proposed workflow, not a report of a shipped feature.

## Name the reference and the behavior separately

The visual reference establishes spacing, hierarchy, typography, and control placement. It does not automatically establish keyboard behavior, network failure handling, or permission checks.

Give the task a stable reference: a specific design frame, an approved image, or a repository asset with a revision. Avoid a link to a design file whose currently selected page can change. Record the viewport and the state shown in that reference.

Then write the interaction contract in plain language. In this example, Escape closes the dialog, focus returns to the button that opened it, and Save remains disabled while a request is pending. A failed request leaves the choices visible and shows an actionable error. None of those outcomes can be established by a single screenshot.

## Prepare the task before handing it off

The product owner prepares the desired outcome and visual reference. An engineer identifies the existing dialog component and the boundaries of the implementation. The runner makes the change with their own authorized development tools and gathers evidence. The design reviewer judges visual alignment; an engineer reviews state handling and accessibility behavior within their expertise. An authorized reviewer records acceptance of the delivered revision.

Teams trying to [manage human and AI work in one system](https://wagglet.com/blog/wagglet-workflow-request-draft-ticket-delivery) need those boundaries to remain visible from preparation through delivery. A result submitted for review should not silently become an accepted or deployed result.

```text
Task: Refresh notification-preferences dialog
Outcome: Match the approved layout without changing saved values.

Reference:
  Design/image: [exact reference and revision]
  Reference viewport: 1440 x 900 CSS pixels
  Narrow viewport: 390 x 844 CSS pixels
  Fixture: standard account, existing preferences loaded

Scope:
  Reuse existing dialog and preference-saving service.
  Change layout and presentation only.
  Escalate changes to API payloads, permissions, or persistence.

Acceptance:
  A1: Heading, controls, and actions follow the approved hierarchy.
  A2: All content and actions are reachable at the narrow viewport.
  A3: Keyboard focus stays within the open modal.
  A4: Escape closes it and restores focus to the opener.
  A5: Pending Save cannot submit a second concurrent request.
  A6: Failed Save preserves edits and exposes the retry action.
  A7: Reopening after success shows the persisted values.

Evidence per criterion:
  Candidate commit: [fill after implementation]
  Environment and browser: [actual versions]
  Steps and fixture: [reproducible setup]
  Observed result: [pass / fail / not checked]
  Attachment or test output: [actual evidence]

Verdict:
  Product/design review: [reviewer, result, conditions]
  Technical review: [reviewer, result, conditions]
  Final decision: [accept / rework / needs review]
  Accepted commit: [exact revision]
  Release approval: separate, not implied
```

## Make the test plan reveal different failures

Start with the normal path: open the dialog, change one preference, save, close, and reopen. That establishes the basic connection between the interface and persisted state.

Next, exercise the boundaries. Use a long notification label. Narrow the viewport. Increase browser zoom. Navigate without a pointer. Delay the save response and try activating Save again. Return an error and confirm that the selected choices remain available for a retry.

These checks answer different questions. A screenshot helps compare visual hierarchy. An interaction recording can show focus behavior. A focused automated test can show that a pending request prevents another submission. No single artifact replaces all three.

Record what actually happened, including an untested case. A list of intended checks is a plan; it becomes evidence only after someone performs them against an identified build.

## Review the exceptions before writing the verdict

Suppose the narrow layout looks correct, but a delayed response allows a second save. The review should identify A5, describe the reproduction, and ask for a correction on the same task. “Almost done” gives the runner less information than “second keyboard activation submitted another request while the first was pending.”

After the fix, request new evidence for A5 and any affected behavior. If the correction changes shared dialog code, broaden review to the other consumers of that component. That is a concrete reason to expand testing; it is not an excuse to repeat every unrelated check indefinitely.

## Keep acceptance attached to the revision

The final verdict should identify the exact revision and any conditions. A reviewer can accept the UI implementation while leaving deployment to the release owner. If someone changes the implementation afterward, the previous verdict does not automatically cover those edits.

This template does not guarantee accessibility, eliminate design judgment, or prove every browser behaves identically. It gives reviewers a common object to inspect. The useful outcome is a decision that another teammate can reconstruct: which behavior was requested, which evidence was observed, who judged it, and what remains outside that judgment.
