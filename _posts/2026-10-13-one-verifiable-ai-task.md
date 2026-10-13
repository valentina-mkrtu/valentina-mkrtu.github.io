---
layout: post
title: 'Turn “make it better” into one task a reviewer can accept'
description: 'A worked example that converts a vague search request into a bounded change with observable acceptance checks.'
date: 2026-10-13
published: false
---

AI disclosure: This article was generated with AI. The search interface below is a hypothetical example, not a report of a shipped change.

## Begin with a decision the reviewer can make

“Make the search better” gives an assistant room to change almost anything. It could rewrite ranking, redesign the input, introduce a new library, or add a network service. Each change might look reasonable in isolation while missing the user's actual complaint.

Replace the request with a decision that someone can make after inspecting the result. For a small task list, that decision might be whether a search for a visible title returns the same records regardless of letter case and accidental spaces at the ends of the query. This is narrow enough to test without claiming that the entire search experience has improved.

The preparer should describe the observed problem before proposing an implementation. In our hypothetical interface, a user pastes “  Reward Review  ” into the filter and gets no results even though “Reward review” appears in the list. That example identifies an input, an observable output, and a likely scope.

## Write the human brief

The product owner prepares the task with an engineer who understands the current filter. Their brief can be short:

```text
Outcome:
  Search the existing title field without sensitivity to letter case
  or whitespace at the beginning and end of the query.

Scope:
  Existing client-side title filter and its focused tests.

Preserve:
  Record ordering, selected item, title display and empty-query behavior.

Exclude:
  Fuzzy search, stemming, translation, new dependencies and server changes.

Executor:
  Assigned engineer or supervised assistant using its own authorized tools.

Reviewer:
  Product owner accepts behavior; engineer reviews implementation.
```

The preserved behavior matters as much as the new behavior. A filter that finds the right items but rearranges the user's list can still be a regression. “Empty-query behavior” must refer to an observed baseline. If the existing product has no agreed behavior for an empty query, resolve that ambiguity before implementation.

## Give execution instructions that match the brief

The executor needs repository context in addition to the product outcome. Point to the existing search component, the relevant test command, and the current branch or revision. Ask the executor to inspect the existing implementation and report a scope conflict before rewriting shared infrastructure.

For the example, the implementation instruction is: normalize the query once per filtering operation, compare it with normalized title text, and preserve the original records and their order. Use the project's existing approach for locale handling. If the product requires language-specific case matching, bring that requirement back to the owner instead of quietly selecting a locale.

That separation supports human supervised AI execution: a person prepares the outcome and constraints, an executor works with their own authorized environment, and named reviewers decide whether the result meets the agreement. The [Wagglet article on dual-prompt task design](https://wagglet.com/blog/dual-prompt-human-agent-task-design) is related reading for keeping the human brief and agent instructions aligned.

## Work one example before expanding the scope

Use a tiny fixture with three titles: “Reward review,” “Reward animation,” and “Build notes.” Record their original order. The query “  REWARD REVIEW  ” should return only the first title. The query “reward” should return the first two in their existing order. A query containing only spaces should follow the agreed empty-query behavior.

These checks describe observable behavior. They do not require the reviewer to approve a particular helper name or internal function structure. The executor can use the smallest change compatible with the repository.

Include a check for an unmatched query and for the selected item while filtering. The change should not mutate source titles to lowercase merely because comparison uses normalized text. Verify that the rendered title still uses its original capitalization.

An assistant can draft the patch and tests, but its final report must distinguish checks it actually ran from checks it only recommends. If the browser or test environment is unavailable, report that limitation directly. A plausible test name is not evidence that a test passed.

## Define the stop condition

The task ends when the agreed inputs produce the expected records, the preserved behaviors remain intact, and the reviewers can inspect the diff and validation evidence. It does not automatically expand into typo tolerance, accent folding, or relevance ranking.

If the executor discovers that the application delegates filtering to a remote service, the initial scope may be wrong. Return that finding with the relevant code location and a proposed revised task. Do not smuggle a server migration into a client-side cleanup because the original request sounded broad.

The reviewer should receive a compact record: changed files, the behavior before and after, commands actually run, and remaining limitations. Product acceptance and implementation review can happen separately, but both must point to the same bounded outcome.

## Know what the task does not establish

Case-insensitive matching is not a complete international search strategy. This example leaves language-specific equivalence, Unicode normalization, performance on large datasets, and accessibility of search feedback for separate decisions. Those omissions are honest limits, not reasons to leave the original problem vague.

The useful habit is to keep the request small enough that a reviewer can reject it for a concrete reason. Once this task is accepted, the team can decide whether another improvement is worth doing and give that next change its own evidence and owner.
