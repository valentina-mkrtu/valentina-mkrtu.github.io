---
layout: post
title: "Recover an unfinished agent task when its author is unavailable"
description: "A recovery runbook for separating saved work, verified evidence, and the next safe action before handing a task to a reviewer."
date: 2026-10-05
---

The task says “almost done.” Its author is offline, the agent conversation ends halfway through an explanation, and a branch contains changes nobody else has reviewed. A teammate has time to help, but cannot tell whether to finish the implementation, investigate a failure, or start again.

The first useful action is to reconstruct the task's state. A recovery brief should let someone answer four questions: what exists, what has been checked, what should happen next, and who can accept the result.

The following worked example is hypothetical. It illustrates a recovery process rather than reporting an actual incident or a measured improvement.

## Preserve the work before interpreting it

Imagine an agent-assisted change that adds a search box to an internal asset browser. The original request is available, along with a remote branch and a note saying “filter implemented; keyboard behavior still needs attention.” The author will be unavailable for the rest of the day.

A replacement coordinator records the branch's full commit identifier and reads the diff. They keep any accessible local changes separate from committed work. If the author's laptop contains unsaved edits that nobody can access, those edits are unknown, not recoverable evidence.

Avoid immediately rebasing, cleaning directories, or asking another agent to rewrite the feature. Those actions can obscure the starting point. First preserve what is available using the repository's normal workflow and identify which revision the investigation concerns.

## Separate observations from inherited claims

“Implemented” is a claim. A diff containing a filter function is an observation. A successful check against that exact revision is evidence of a particular behavior.

For the asset-browser example, the coordinator might identify three questions to investigate: whether entering text narrows the list, whether clearing text restores it, and whether keyboard focus remains usable when the list changes. The old note helps choose these questions but does not answer them.

Keep a small evidence ledger. Each entry needs the revision, environment, action performed, observed result, and any uncertainty. A screenshot without the input sequence may show appearance while proving little about keyboard behavior. A green test log from an earlier revision may no longer apply.

## Write the recovery record

This record is a template for the hypothetical asset-browser task. Fill the placeholders from the actual repository and investigation. Unknown values should remain explicitly unknown until someone checks them.

```text
TASK: Asset-browser search
RECOVERY OWNER: <replacement coordinator>
ORIGINAL REQUEST: <task reference>

STATE
Repository: <authorized repository>
Available revision: <full commit identifier>
Preserved local changes: <list, none, or unavailable>
Original author's note: Filter implemented; keyboard behavior uncertain
Verified behavior: Not yet established by the recovery runner

SCOPE
Required: Search existing asset names and clear the query
Boundary: Preserve existing selection and keyboard interactions
Excluded: New indexing service, dependency changes, deployment

EVIDENCE TO COLLECT
Record setup and exact reproduction steps
Test matching, no-match, and cleared-query states
Check keyboard focus after each state change
Attach actual output and identify the tested revision

NEXT ACTION
Inspect existing keyboard conventions and reproduce current behavior
Stop if expected selection behavior is not defined

REVIEWER HANDOFF
Runner: <teammate with their own approved tools and repository access>
Acceptance reviewer: <qualified reviewer>
Delivery: Revision, diff summary, evidence, remaining uncertainty
Acceptance: Pending reviewer decision
```

The next action should be small enough to complete without reconstructing the entire abandoned conversation. “Finish search” is too broad. “Reproduce focus behavior after a no-match query” gives the runner a concrete starting point.

## Choose a runner who can verify the result

The coordinator prepares the recovery brief; the runner investigates and implements within its boundaries; the reviewer judges acceptance. These may be different people, and their responsibilities should remain explicit even on a small team.

The runner uses their own approved agent tools and repository identity. A task assignment does not grant access to another person's account or credentials. If the runner cannot reach the repository, required fixtures, or approved environment, resolve that access gap before proceeding.

AI task handoff for teams works best when the receiving person can recognize the relevant failure. For this example, familiarity with the application's keyboard conventions matters more than familiarity with the original agent conversation. [Wagglet for startups](https://wagglet.com/for/startups) describes routing prepared tasks to available teammates or agents without sharing accounts; the recovery record here supplies a concrete way to prepare that work.

## Decide whether to continue or restart

Continuing is reasonable when the saved changes are understandable, the requirement is stable, and a small reproduction can establish the baseline. Restarting may be justified when the available work cannot be explained or its assumptions contradict the current request.

Record that decision and its reason. Do not quietly throw away partial work merely because a new agent prefers a different structure. Equally, do not keep a patch solely because somebody has already spent time on it.

If the original author returns, give them the recovery record before merging competing changes. Two people independently finishing the same task creates another coordination problem.

## Return evidence to the reviewer

The runner's delivery should identify what changed after recovery, which checks actually ran, and what remains untested. The reviewer then compares that evidence with the agreed scope and decides whether to accept the submitted revision or request further work.

An unavailable author may leave a product decision unresolved. If nobody is authorized to decide whether filtering should clear selection, stop at that decision instead of letting the agent invent expected behavior. A technically plausible implementation can still implement the wrong interaction.

Recovery is successful when the team can explain the current state and make a supported next decision. It does not promise that every abandoned task is worth finishing, or that reconstructing context will be faster than starting over.
