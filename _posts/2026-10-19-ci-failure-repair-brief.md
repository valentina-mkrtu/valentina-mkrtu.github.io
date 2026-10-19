---
layout: post
title: "A failing CI job needs a reproduction before it needs a bigger prompt"
description: "A failing CI job needs a reproduction before it needs a bigger prompt. A practical implementation and review guide."
date: 2026-10-19
---

This article was written with AI assistance as part of Wagglet marketing. This tutorial uses an intentionally broken local fixture, not a customer incident.

A red CI job invites a broad request: “Fix the build.” That request is usually too loose. It does not say whether the failure is in application behavior, the test environment, dependency installation, or the job definition. A coding agent can make the status green by changing the wrong thing if the brief never explains what the check protects.

A better starting point is a small reproduction with a bounded repair. The human runner establishes the failing state. The task author states the behavior that must remain true. The agent investigates inside that boundary, and a reviewer evaluates the evidence from the resulting revision.

## Reproduce a failure you can explain

Here is a tiny dependency-free example using Node's built-in assertion module. Save it as `ci-repro.cjs` and run `node ci-repro.cjs`. The initial implementation is deliberately wrong for whitespace-only input.

```js
const assert = require('node:assert/strict');
function isBlank(value) {
  return value === '';
}
assert.equal(isBlank(''), true);
assert.equal(isBlank('   '), true, 'spaces should count as blank');
assert.equal(isBlank('hello'), false);
```

The second assertion should fail. The exact stack formatting depends on the installed Node version; the important evidence is the failed expectation and nonzero exit status. Record the runtime version, command, and actual output from your environment rather than pasting an assumed transcript into a delivery report.

The repair for this fixture is narrow: accept string inputs whose trimmed length is zero. That does not authorize changing callers, coercing arbitrary values, or rewriting validation across an application. Add a tab-and-newline case and a nonempty string with surrounding whitespace so the intended boundary is visible.

## Write a brief that protects the test

The most important part of the repair brief is what the agent must preserve. If it can remove the assertion or exclude the test from CI, a passing job becomes a poor signal.

```text
Outcome: isBlank returns true for empty or whitespace-only strings.
Input contract: strings only; do not add coercion for other types.
Allowed change: implementation and focused regression assertions.
Preserve: existing nonempty-string behavior and the CI test command.
Do not: skip tests, suppress assertion failures, or alter job success rules.
Stop if: the failure cannot be reproduced with the recorded runtime.
Delivery: failing baseline, corrected run, diff, revision, and limits.
```

In a real repository, replace the file and command with verified paths. Include the dependency lockfile and relevant environment details if the reproduction depends on them. Do not paste unrelated CI logs simply because they are available; they make it harder to identify the assertion that matters.

## Avoid three common detours

The first detour is treating a missing dependency as a behavior defect. If installation fails, the application test may never have run. Record that distinction before asking for a code change. A runner should not describe a setup failure as a reproduced regression.

The second is accepting a local pass without matching the CI invocation. A local command may select fewer tests, use a different runtime, or rely on files that are ignored by version control. Run the relevant CI command from the intended revision and inspect whether the required files are actually included.

The third is changing the test expectation to match the bug. Sometimes a test is wrong, but that requires a reason tied to the acceptance contract. The reviewer should be able to see whether the implementation changed, the expected behavior changed, or both. Those are different decisions with different owners.

## Give the runner a concrete job

Human supervised AI execution works best when the human's contribution is observable. Here, the runner confirms the workspace, reproduces the baseline failure, executes the approved checks, and reports unexpected access or setup problems. They use their own authorized tools and do not borrow another person's account to get past a blocked step.

The author prepares the task and its scope. The agent can propose the implementation and help run checks within that scope. A qualified reviewer checks the diff, the regression cases, and the evidence before accepting the delivery. Deployment remains a separate decision.

[Wagglet for engineering teams](https://wagglet.com/for/engineering-teams) describes keeping the brief, execution context, delivery evidence, and review attached to the same task. That organization is useful only if the evidence itself is specific enough to inspect.

## Review the smallest complete evidence packet

For this fixture, the packet contains the original failed assertion, the corrected function, the extra whitespace cases, and a successful rerun from the recorded environment. In a larger project, it should also identify the tested commit and distinguish focused checks from broader checks that were not run.

Before acceptance, verify that the failure existed before the change, the repair addresses the stated behavior, and the test still fails if the repair is removed. Check that no unrelated behavior was changed to satisfy the fixture. Finally, record any untested integration or platform cases.

This example does not prove anything about an actual production CI job. It demonstrates a repair workflow that keeps the problem small enough to explain. A useful delivery lets the reviewer understand why the job changed from failing to passing, rather than asking them to trust the color of a status badge.
