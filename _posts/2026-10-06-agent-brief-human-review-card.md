---
layout: post
title: "Write the agent brief and the human review card separately"
description: "A two-part task example for adding an archive confirmation, with explicit pause points and acceptance evidence."
date: 2026-10-06
---

The example is a proposed workflow, not a report from a customer project.

A task can be detailed enough for a coding agent and still leave its human supervisor unsure what to do. The agent gets file paths, requirements, and constraints. The person sees a long technical prompt and assumes that checking the final screenshot will be enough.

That gap matters when a change touches product behavior. Consider a small example: adding a confirmation dialog before a user archives a project. The implementation may be straightforward, but someone still has to decide what the dialog promises, who is allowed to archive, and what evidence counts as a successful change.

Writing separate instructions for the agent and the supervising person gives those decisions an owner. Both parts belong to the same task, refer to the same outcome, and meet at the same review.

## Start with a behavior people can inspect

The desired outcome in this fictional example is precise: an authorized user can open the archive dialog, cancel without changing the project, or confirm the existing archive operation once. The task does not introduce permanent deletion, change permissions, or redefine what archiving means.

The task author prepares this boundary before another person starts the work. If existing behavior is undocumented, discovering it is part of preparation. A sentence in a prompt should not quietly become a new product policy.

The runner then uses their own authorized development environment and AI tools. The reviewer is the person responsible for judging whether the delivered change satisfies the task. Those roles may overlap in a small team, but the responsibilities still need to be written down.

## Part one: the agent's implementation brief

The agent needs enough context to find the existing path, preserve its constraints, and produce reviewable evidence. Here is a reusable example; repository-specific paths and commands must be filled from the actual project.

```text
Outcome
Add a confirmation step before the existing project archive action.

Starting point
Use the repository, base revision, and dialog component named in
this ticket. Inspect the current archive handler before editing.

Allowed changes
Dialog presentation, its connection to the existing handler, and
focused tests for confirmation, cancellation, and repeated input.

Behavior
Cancel leaves the project unchanged.
Confirm calls the existing archive action once.
While the request is pending, repeated confirmation is disabled.
An error keeps a recoverable UI and does not claim success.

Pause and escalate
Stop if the archive endpoint also deletes data, if the permission
check is unclear, or if existing copy contradicts observed behavior.
Do not select new product semantics to unblock implementation.

Delivery
Identify the exact revision, changed files, tests and their actual
results, remaining limitations, and the steps for human inspection.
```

This brief names the smallest useful implementation surface. It does not prescribe an imagined component API or assume a particular framework. The runner should replace the starting-point references with verified project details before handing it to an agent.

## Part two: the human supervision card

The human instructions should focus on decisions and observations. Repeating the implementation brief in shorter prose hides the reason for having two parts.

```text
Before starting
Use the agreed test workspace with synthetic projects. Confirm
that your account may perform the archive action there.

During the run
If the agent flags deletion behavior or unclear permissions,
pause the task and ask the product owner. Do not approve a guess.

Inspect the delivered revision
Open the dialog and cancel: the project should remain active.
Open it again and confirm: check the resulting project state.
Repeat confirmation quickly: inspect whether requests duplicate.
Trigger the agreed error fixture: inspect the message and recovery.
Use keyboard navigation and verify focus returns appropriately.

Record
Revision inspected, environment, each observation, and unresolved
questions. Attach genuine evidence from this attempt.

Acceptance
The named reviewer checks the code and evidence before acceptance.
Successful delivery alone does not authorize a production release.
```

The reviewer can now distinguish missing evidence from a failed implementation. If keyboard behavior was not inspected, the record should say so. An empty checklist is a request for work, not a pass.

## Connect escalation to a real decision

Suppose the runner discovers that archiving also removes a project from a shared dashboard immediately. The dialog copy only says that the item leaves the current user's list. The agent cannot repair that mismatch merely by choosing clearer wording: the team first needs to agree on the behavior being promised.

The runner pauses and records the contradiction. The task author resolves it with the responsible product person, updates the shared outcome, and identifies any new evidence needed. Both the implementation brief and the supervision card must reflect the resolution before work resumes.

This keeps escalation useful. It is not an instruction to ask about every ordinary coding choice. It names the decisions that could invalidate the task itself.

## Keep the two parts attached to one outcome

Teams trying to manage human and AI work in one system need a shared record of the result without forcing every participant to read the same instructions. [Wagglet's Dual Prompt article](https://wagglet.com/blog/dual-prompt-human-agent-task-design) describes that separation between agent context and human supervision.

For the archive example, the record should connect the prepared brief, the runner's attempt, the inspected revision, and the review decision. No account sharing is needed to preserve that connection. Each participant uses their own authorized access, and evidence moves with the task.

## Where this pattern ends

Separate instructions do not guarantee that the agent follows them or that the human notices a defect. They make responsibilities easier to inspect. Tests, access controls, and informed review still carry the actual burden of checking the work.

Start with one task whose outcome is easy to observe. If the agent cannot tell when to pause, sharpen its brief. If the supervisor cannot tell what to inspect, sharpen the review card. The task is ready when both participants can explain their next action without borrowing the other's role.
