---
layout: post
title: "Five mistakes that make a task history impossible to audit"
description: "A worked example and a compact checklist for preserving the evidence between a request and an accepted delivery."
date: 2026-10-14
---

A task history should explain how a request became an accepted result. That sounds straightforward until a reviewer opens a finished task and finds a vague description, three overwritten delivery notes, and a screenshot with no revision attached. There is plenty of activity, but little evidence connecting it.

The repair is usually small. Keep the original request, identify each attempt, attach evidence to a specific candidate, and record who made the acceptance decision. These habits matter even when the change itself is only a few lines.

## Mistake 1: treating the request as a complete specification

“The empty state is broken” is a useful observation. It is not enough to tell an executor which behavior to implement. The task author needs to identify the affected screen, the conditions that produce the problem, and the expected result.

For an illustrative example, suppose an account with no saved reports sees a loading spinner forever. Preparation could define the result this way: after the reports request returns an empty array, show the empty-state message and a working create-report action. Do not change the error state or the loading state while the request is still pending.

That distinction gives the reviewer three observable conditions. It also prevents a superficially successful fix that removes the spinner before the request has completed. The author owns this preparation; the executor should not have to infer the product decision from a screenshot.

## Mistake 2: losing the claim boundary

A claim identifies who owns an attempt. Without it, two people can start competing changes, or everyone can assume someone else is investigating. Record the runner and when responsibility began.

The runner works through their own authorized tools and permissions. If a required environment is unavailable, record that blocker and ask the responsible owner to resolve it. A task assignment does not grant new repository permissions or authorize use of another person's account.

For work management for human and AI teams, this ownership boundary remains useful even when much of the implementation is delegated. Somebody still owns the attempt and has to surface missing access, ambiguous requirements, and the actual result.

## Mistake 3: attaching evidence to a task instead of a candidate

A screenshot named “fixed.png” is weak evidence if the task contains several candidate revisions. Name the revision, environment, and relevant input beside the screenshot or test output. Evidence should answer what was checked, not merely suggest that checking happened.

In the empty-state example, a result from yesterday's build cannot validate today's change. If the candidate changes after testing, repeat the affected checks. Preserve the old result as part of the earlier attempt rather than overwriting it and making the history look cleaner than it was.

This does not require copying every terminal line into the ticket. A concise evidence index is easier to use. Point each acceptance criterion at a stable test result or observable artifact and call out any check that could not run.

## Mistake 4: collapsing delivery into acceptance

Here is a fictional timeline for the example. The times illustrate recordkeeping; they are not measured work or a customer case.

| Time | Event | Record to retain |
| --- | --- | --- |
| 09:00 | Request received | Empty account keeps spinning |
| 09:15 | Preparation complete | Empty, pending, and error criteria |
| 09:25 | Runner claims | Executor and authorized environment |
| 10:10 | First delivery | Candidate A and its evidence |
| 10:25 | Reviewer sends back | Error state changed unintentionally |
| 10:50 | Second delivery | Candidate B and fresh checks |
| 11:05 | Reviewer accepts | Criterion-by-criterion verdict |

The first delivery is useful even though it was not accepted. It shows where the implementation departed from the scope. The second delivery should explain the correction so that the reviewer can focus on the relevant difference without losing the broader regression checks.

[Wagglet's workflow description](https://wagglet.com/blog/wagglet-workflow-request-draft-ticket-delivery) gives a concrete example of keeping preparation, claim, delivery, and reviewer verdicts separate. The same distinction can be implemented in a small repository's issue template. What matters is that the record preserves the decisions instead of turning every transition into an ambiguous “done.”

## Mistake 5: assuming acceptance proves release

Acceptance says the reviewer accepted a result against the task's criteria. It does not by itself establish that a branch was merged or that a release reached users. Record those facts separately when they are in scope, with evidence from the repository and release process.

For the empty-state fix, the acceptance note can truthfully say that candidate B meets the criteria while deployment remains unverified. That is more useful to support staff than a finished card that implies the issue has disappeared for everyone.

## A review checklist for the next small fix

Before accepting a task, check whether another teammate could reconstruct the following without asking the original runner to remember it:

- The original request and the prepared acceptance criteria.
- The person responsible for each execution attempt.
- The exact candidate that the evidence describes.
- The result of each required check, including checks that could not run.
- The reviewer and the reason for acceptance or rework.
- Separate merge and deployment evidence, if either is required.

The checklist does not prove correctness. A test can be poorly designed, a screenshot can omit the failure, and a reviewer can miss a regression. Its purpose is to make those judgments inspectable. When a task returns later, the team can begin with a record of what was actually checked instead of rebuilding the whole conversation from memory.
