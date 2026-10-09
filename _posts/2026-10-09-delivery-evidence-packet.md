---
layout: post
published: true
title: "A technical delivery should be an evidence packet, not a victory message"
description: "How to connect changed files, test output, screenshots, and known limits to a reviewable acceptance decision."
date: 2026-10-09
---

“Implemented and tested” is a summary, not evidence. A reviewer still needs to know which revision was tested, what the test exercised, what changed, and which part of the request remains uncertain. Without those details, acceptance becomes a second investigation.

An evidence packet makes the relationship between the requirement and the delivered artifacts explicit. It does not need to be long. It needs to answer the reviewer’s questions without asking them to trust a confident description of work they cannot inspect.

This article uses a hypothetical search-filter repair. No code changes, screenshots, or test executions are reported as real results. The example shows how to structure the delivery once those artifacts exist.

## Start with the acceptance claim

Suppose the request says that a selected category must remain selected after the results refresh. The delivery’s main claim should be equally specific: “The selected category is preserved across a successful refresh.” That claim can be checked against a fixture and an interaction sequence.

Do not broaden it to “filtering is fixed” unless the evidence covers every filtering behavior. Likewise, distinguish a successful refresh from an error response or a navigation event. If those cases are outside scope, name them instead of allowing a screenshot to imply universal coverage.

The request owner prepares the expected behavior. An engineer supplies the technical context and test boundaries. The runner executes with their own authorized tools and returns artifacts. The named reviewer decides whether those artifacts satisfy the request. Evidence helps that decision; it does not transfer acceptance authority to the runner.

## Attach a test result that can be interpreted

Record the exact command, relevant environment, revision, and output. A line saying “tests passed” omits whether the test reached the failure that motivated the task. A large terminal dump can hide the same problem under more text.

Use a focused summary with the original output attached. For a regression test, state whether it was checked against the earlier behavior. If that check was not performed, do not imply that the test is proven to catch the original bug.

```text
Acceptance criterion:
Selected category remains selected after results refresh.

Test evidence:
Revision: [actual tested revision]
Command: [exact executed command]
Environment: [runtime and relevant dependency versions]
Observed result: [actual output and exit status]
Regression sensitivity: [observed result on earlier revision, or not checked]

Interpretation:
[Which criterion this supports and which cases it does not cover]
```

The interpretation should be shorter than the evidence. Its purpose is to connect the result to the requirement, not to rewrite a failure into a success.

## Use a screenshot for a visual claim

A screenshot can show that the category label remains visible after refresh. It cannot prove that the underlying query used that category, or that the state survives a later navigation. Pair visual proof with the appropriate state or behavior check.

Capture the same fixture and viewport before and after the repair. Include the selected category and enough of the results view to make the relationship understandable. Remove private data and irrelevant browser chrome without cropping away the context needed to judge the result.

The artifact slot for this example is: “[Actual redacted capture showing the selected category after refresh; include tested revision and capture conditions.]” It is a request for evidence, not a description of a screenshot already produced.

## Explain changed files by responsibility

List the actual changed paths with a short reason for each. A reviewer should be able to distinguish the implementation, the regression test, and any supporting fixture. Do not invent filenames to make an incomplete delivery appear concrete.

For the hypothetical repair, the packet might identify the filter-state owner, the refresh handler, and the regression test. If a shared component changed, explain the broader surface area. If a lockfile changed, state why rather than letting it disappear into a generic file count.

Keep the packet tied to the revision being reviewed. Evidence from an earlier iteration may remain useful history, but it must not masquerade as verification of the latest patch.

## Give limitations a visible place

Useful limits are specific: “Error-response behavior was not exercised,” “Only the desktop viewport was captured,” or “The test uses a fixture rather than the production search service.” These statements tell the reviewer what remains unknown.

Avoid “No known issues” when the actual meaning is that no additional cases were checked. If the limitation prevents acceptance, return to the task. If the reviewer accepts it, record that decision and any follow-up responsibility explicitly.

For managing technical deliveries, the key connection is between the original request and the evidence returned for it. [Wagglet’s request-to-delivery workflow](https://wagglet.com/blog/wagglet-workflow-request-draft-ticket-delivery) describes that lifecycle; the packet structure here can be applied wherever the team records its work.

## Let the reviewer make a bounded decision

The final review should answer whether the accepted revision satisfies the stated criteria, whether the evidence is sufficient, and whether any remaining limitation is acceptable. Acceptance does not silently authorize deployment, new scope, or access to another environment.

A strong delivery makes disagreement easier to resolve. The reviewer can point to a missing case or artifact instead of asking the runner to “check everything again.” The next iteration then has a clear purpose: add the evidence or correction needed for the original decision.
