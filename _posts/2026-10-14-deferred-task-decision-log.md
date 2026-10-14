---
layout: post
title: "A deferred AI task needs a useful decision log"
description: "How to record missing access, unclear scope, and a better owner without losing the work already done."
date: 2026-10-14
published: true
---

A task that stops can leave the next person with either a useful diagnosis or a blank status. “Blocked” describes a condition. It does not explain what happened, who can change it, or whether the original request should still be pursued.

This matters when work moves between a person who prepares a request and another person who executes it with an agent. The second person may discover a missing permission, an unresolved product decision, or a need for specialist judgment. Those are different reasons to stop. A decision log should preserve the distinction.

The example below is hypothetical. It is a suggested format for transferable AI work, not a report about a real incident or a claim that a particular tool automatically diagnoses blockers.

## Distinguish deferred from declined

Deferred means the task could proceed when a stated condition changes. For example, a test environment may be unavailable until its owner restores it. The record should name that condition and the person responsible for checking it.

Declined means the proposed assignment is unsuitable in its current form. Perhaps the runner cannot assess the consequences of a database migration, or the requested behavior conflicts with a product decision. The next action might be reassignment, rescoping, or closing the request. It should not default to an endless sequence of retries.

Neither label is a judgment about a teammate’s effort. The purpose is to describe the work accurately enough that someone can make the next decision.

## Capture the last verified point

Suppose a team requests a correction to its export flow. The author provides a repository reference and asks the runner to reproduce duplicate requests, implement a bounded fix, and attach evidence for review.

The runner starts with their own authorized tools and discovers three things. They can read the repository. They can run the local application. They cannot access the staging dataset described in the brief. They also find that “duplicate” could refer to repeated clicks or two background jobs with the same identifier.

A useful log says exactly that. It does not say the environment is entirely broken, and it does not claim that the agent proved a server defect. The verified point is that the local application starts; the relevant production-like reproduction remains unavailable.

## Use a decision record

```text
Task: prevent unintended duplicate exports
Decision: deferred before implementation
Last verified point: local application starts from the supplied revision

Missing access:
  Staging dataset in the brief is unavailable to the assigned runner
  Resolution owner: staging environment owner
  Resume when: owner supplies an authorized test fixture or grants scoped access

Unclear scope:
  Repeated UI activation and duplicate background jobs are distinct failures
  Decision owner: export feature owner
  Resume when: one failure and its acceptance checks are selected

Alternate owner:
  If the defect is job deduplication, route investigation to the backend maintainer
  Proposed runner remains available for the bounded UI case

Evidence preserved:
  Revision, setup steps, local startup outcome, and unresolved questions
Next checkpoint:
  Author reviews the fixture and scope decision before the task returns to ready
```

This format avoids copying private datasets or credentials into a broadly visible task. Refer to approved locations and the responsible access owner. A transfer of work does not transfer another person’s account or expand the runner’s permissions.

## Give each participant a next action

The author owns preparation. In this example, they need to select the failure the task is meant to address and supply acceptance criteria for it. They should also confirm that the request is still worth doing after the new information arrives.

The runner owns the execution record. They preserve what they checked, what they could not check, and why they stopped. They do not need to produce a speculative patch to prove that time was spent.

The reviewer owns acceptance. Before work resumes, the reviewer can say whether the proposed fixture and checks would be sufficient to judge a result. If the evidence would still be inconclusive, the task is not ready.

The alternate owner is a proposal until that person accepts the assignment. Writing someone’s name into a log does not create their consent, available time, or authorization. The author should confirm the handoff through the team’s normal process.

## Resume from evidence, not memory

When the missing fixture arrives, revisit the stop record. Confirm that it resolves the actual blocker and that the scope has not quietly expanded. A new dataset may enable the reproduction while revealing that a different system owns the defect.

If the task resumes, retain the earlier decision and append the new evidence. Replacing “blocked” with “in progress” without preserving the reason erases information that could improve future task preparation. A short history also helps the reviewer understand why a proposed implementation differs from the first request.

The [Wagglet workflow from request through delivery](https://wagglet.com/blog/wagglet-workflow-request-draft-ticket-delivery) provides context for keeping preparation and acceptance distinct. This decision log is an independent working example of the information a team can preserve at those boundaries.

## Keep the log proportionate

A trivial missing file may need only two sentences. A consequential migration may need a specialist review before anyone decides how to continue. Use the detail required to support the next decision, rather than turning every pause into a formal incident report.

The practical check is simple: could another authorized teammate read the record and explain what must change before execution begins again? If the answer is no, add the missing condition or owner. If the answer is yes, leave the work deferred until that condition is met.

AI disclosure: AI assisted with this article. No real team performance or incident outcome is asserted.
