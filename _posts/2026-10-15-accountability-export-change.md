---
layout: post
title: "Accountability gaps in an agent-written export change"
description: "A worked export-change example with explicit preparation, execution, review, and release responsibilities."
date: 2026-10-15
---

A change can have an author, an executor, a reviewer, and a release owner while still fall between all four roles. The gap appears when each person assumes another person made a decision that was never recorded. An agent writing the implementation does not remove that gap. It can make the gap harder to see by producing a polished completion summary.

Consider a hypothetical tool that exports a level summary as CSV. A designer asks for an additional difficulty column. The request sounds small, but the tool has existing consumers, an established column order, and a release process. The useful AI task handoff for teams is a decision record that makes those boundaries inspectable.

## Mistake 1: assigning a result without assigning its meaning

The task author owns the requested outcome. They specify whether the new column is always present or enabled by an option, what an empty difficulty value means, and whether existing consumers must continue accepting the default export. Those are requirements, not details the implementation agent should infer.

The author supplies a synthetic input and expected output. For example, a level called Meadow has difficulty Normal, while a second level has no assigned difficulty. The sample should show both rows and the intended representation of the missing value. If the team has not decided between an empty field and a literal word, that uncertainty belongs in the brief.

A common failure is to ask the runner to “use judgment” without identifying whose judgment governs compatibility. The runner can identify the ambiguity and propose alternatives. An authorized domain owner must decide which behavior is correct before the patch is accepted.

## Mistake 2: treating execution access as release authority

The runner uses their own authorized repository and agent tools to make a bounded attempt. They confirm the base revision, inspect existing work, run the sample, and collect evidence. Their ability to edit the repository does not automatically authorize changing production export settings or deploying the tool.

The first implementation should stay within the agreed files and behavior. If adding the column reveals an old parser defect, the runner records it and checks whether it blocks the requested result. They should not silently turn a CSV addition into a rewrite of every export consumer.

The agent can generate code and suggest tests. The runner remains responsible for distinguishing suggestions from executed checks. A proposed test in a chat is not a passing test, and a successful command against an earlier revision does not validate the final patch.

## Mistake 3: hiding two reviews behind one approval

The designer can judge whether the exported difficulty represents the level data correctly. A qualified engineer judges escaping, column compatibility, error handling, and the implementation. If one person is qualified for both, the record can say so. If not, use two explicit acceptance checks.

```text
Author: tool owner; defines optional-column behavior and sample outputs
Runner: authorized teammate; implements and records actual checks
Domain reviewer: designer; checks meaning of normal and missing values
Code reviewer: engineer; checks format, compatibility, and failure paths
Deployer: release owner; ships the accepted revision through normal process
Evidence: input fixtures, generated CSVs, diff, commands, exit codes
Stop: consumer contract changes or unclear missing-value requirement
```

This map is a template, not a claim that named people have reviewed a real change. Replace the roles with actual owners and attach the evidence from the actual attempt.

The [Wagglet discussion of meaningful work with human–AI pairs](https://wagglet.com/blog/meaningful-work-with-human-ai-pairs) helps frame the division of responsibility. A prepared task and an agent-assisted implementation still need a human decision about whether the result is acceptable.

## Mistake 4: letting a happy-path file become the whole verdict

A single readable CSV establishes very little. Review the default export, the enabled option, an empty difficulty, a value containing punctuation, and the consumer behavior that the task promises to preserve. Use the same fixtures before and after where comparison is meaningful.

The reviewer should be able to connect each artifact to the final revision. Include the exact command used to generate it. If the environment could not run a consumer compatibility check, say which check is missing and why. An explicit limitation supports a release decision better than an unqualified “done.”

Do not inflate acceptance into an unlimited checklist. The task author and reviewer should agree which cases are relevant to this change. The point is enough evidence for the actual contract, not a collection of unrelated green checks that looks reassuring.

## Mistake 5: treating acceptance and deployment as one event

Acceptance means the named reviewers judged the delivered revision against the brief. Deployment means an authorized release owner chose to put that revision into the target environment. Those events can happen close together, but they answer different questions.

The deployer checks the release identifier, expected consumer impact, and recovery path. They should not rebuild an unreviewed local variant and assume the earlier verdict transfers. If a release requires configuration changes outside the task, those changes need their own explicit authorization and verification.

The record is complete when a later teammate can identify the requested behavior, the implemented revision, the evidence, the acceptance owners, and the shipped revision. This does not guarantee defect-free software. It prevents accountability from disappearing into the space between a plausible agent response and an actual release.
