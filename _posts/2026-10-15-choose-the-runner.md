---
layout: post
title: "Choose the runner before choosing the AI tool"
description: "A practical decision record for handing a bounded task to a teammate with the right access and judgment."
date: 2026-10-15
---

A startup can have several capable AI tools and still struggle to assign a small technical task. One teammate has a terminal agent, another uses an editor assistant, and a third understands the customer workflow but does not maintain the application. Picking whoever has the newest model skips the questions that determine whether the work can actually finish.

The runner needs authorized access to the relevant environment, enough judgment to recognize the requested outcome, and a tool that can perform the bounded operation. Those requirements overlap, but they are not interchangeable. A strong coding model cannot grant repository access. Repository access does not establish permission to deploy. Product knowledge does not automatically qualify someone to approve a technical change.

The example below is a proposed workflow, not a report of a customer result or a measured productivity gain.

## Start with the evidence the task must produce

Consider a hypothetical staging bug: changing a notification preference saves the value, but the confirmation message shows the previous selection. The backend receives the correct value. The requested outcome is a confirmation message that reflects the completed save, including after a failed request and retry.

An engineer prepares the task. They supply the reproduction sequence, the relevant component, a synthetic account, the supported local setup, and the boundary: no changes to authorization, storage, or the server contract. They also identify the reviewer who will inspect the diff and the test results.

The runner's job is to use their own authorized tools to reproduce the problem, supervise the bounded implementation, and record what happened. The reviewer decides whether the evidence supports acceptance. Keeping those jobs explicit prevents an agent's final message from becoming the acceptance decision by default.

## Compare candidates against the actual task

Suppose Maya understands the preference flow and has staging access, but cannot access the repository. Leo has repository access and a terminal agent, but has never checked this workflow. Noor has both approved repository access and staging access, and has previously tested notification settings. Their editor assistant can edit files and run the project's documented checks.

Noor is the strongest starting candidate for this particular task. The decision follows from access and relevant judgment, not from a ranking of people or model brands. Maya can still help clarify the observed behavior. Leo might prepare setup instructions or become the runner for a different, more technical task.

If Noor's tool cannot run the required test command, that limitation becomes visible before implementation. The team can choose another eligible runner or arrange an authorized test environment. It should not solve the problem by copying another person's login, session, API token, or agent account.

## Write a decision record short enough to maintain

The useful record explains why this assignment is workable and what would invalidate it. It does not need to become a scoring system with misleading precision.

```text
Outcome: confirmation text matches the preference saved in staging
Prepared by: component owner
Runner: Noor, using her own authorized editor and repository access
Reviewer: an engineer who did not perform the change

Access confirmed: repository, synthetic staging account, local test runner
Relevant judgment: understands the notification preference flow
Tool requirement: inspect files, make a narrow edit, run documented tests
Evidence: reproduction before/after, commit reference, commands and outputs
Stop: server contract change, missing access, unrelated failure, unclear state
Acceptance: reviewer inspects evidence against the named cases
```

For a team asking how to **delegate AI task without sharing account** access, the useful unit is this bounded assignment. The [Wagglet article on AI task handoffs](https://wagglet.com/blog/close-team-skill-gaps-with-ai-task-handoffs) describes how expert preparation and a concrete human check can support that arrangement. The handoff carries context and acceptance criteria; the runner supplies their own authorized execution environment.

## Make the first checkpoint a readiness check

Before editing, Noor confirms the exact repository and base revision, checks for existing work, and reproduces the issue using synthetic data. If the reported behavior cannot be reproduced, the deliverable is a precise discrepancy: which steps ran, what appeared, and which environment differed. Guessing at a fix would weaken the evidence.

Once reproduction succeeds, the agent can propose the smallest relevant change. Noor checks the visible workflow while the engineer later checks implementation details. The evidence should distinguish a test that ran successfully from a test that was skipped, unavailable, or unrelated. A screenshot of a correct message is useful, but cannot prove that failure recovery works.

## Keep tool differences out of the acceptance contract

The author should specify outcomes and project-supported commands rather than require a teammate to imitate another person's chat history. Different tools may need different prompts or ways of loading context. The acceptance cases remain the same: successful save, failed save, retry, and a second change after completion.

This is also where limits matter. A task that expands into permissions, production data, or an unfamiliar server behavior needs a new assignment decision. The original runner was selected for a small confirmation-state defect. That selection is not a standing authorization for every issue discovered along the way.

The final handoff contains the changed revision, observations, test output, remaining uncertainty, and the named reviewer. That record lets the reviewer accept the result, request a specific correction, or return the task without reconstructing an entire agent conversation.
