---
name: requesting-code-review
description: Use when a completed change set needs review before integration or handoff
---

# Requesting Code Review

Dispatch a review subagent (`superpowers:code-reviewer-light` or `superpowers:code-reviewer`) to catch issues before merge or handoff. The reviewer gets precisely crafted context for evaluation — never your session's history. This keeps the reviewer focused on the work product, not your thought process, and preserves your own context for continued work.

**Core principle:** Review the completed integrated result at meaningful checkpoints, not every task by default.

## When to Request Review

**Required for Deep Formal Review:**
- After Full Lane work in `executing-plans`
- After all tasks in subagent-driven development
- After all waves in parallel subagent execution
- Before merge when Full Lane review triggers apply

**Optional but valuable:**
- When stuck (fresh perspective)
- Before refactoring (baseline check)
- After fixing complex bug

Routine per-task formal review is not the default workflow.

## Review Timing

- Standard Deep Formal Review timing: request review once after all implementation tasks are complete and the integrated result has been verified.
- Review the completed branch or task series after integration — not a routine per-task checkpoint.
- Use `superpowers:receiving-code-review` only if a light or deep review returns feedback that needs technical evaluation.

## Review Threshold

Outside workflows that already require a final formal review, request review when one or more apply:
- the user explicitly asks for review
- 5 or more files changed
- persistence, schema, or storage logic changed
- shared abstractions or architecture changed
- there is significant integration or regression risk

Do not request review by default for:
- small local bug fixes
- isolated display or formatting changes
- copy-only changes

## Review Lanes

- **Light Review** - for small, local, low-risk changes when review is useful but a deep formal review is unnecessary
- **Deep Formal Review** - for Full Lane work and for changes with architecture, contract, persistence, or meaningful regression risk

## Reviewer Selection

Use `superpowers:code-reviewer-light` when all of the following are true:
- the review scope is 1-2 files or one narrow local area
- the change does not touch persistence, auth, migrations, or shared contracts
- the goal is local correctness, requirement match, and touched-test sufficiency

Use `superpowers:code-reviewer` when any of the following are true:
- the work is Full Lane
- the review scope spans multiple files or an integrated task series
- architecture, shared abstractions, public contracts, persistence, auth, or migration behavior changed
- the user explicitly asks for a formal review

## How to Request

**1. Get git SHAs:**
```bash
# Default: review the completed integrated result across the full branch or task series
BASE_SHA=$(git merge-base HEAD origin/main)  # or origin/master / actual base branch
HEAD_SHA=$(git rev-parse HEAD)

# Only for a single-commit spot check:
# BASE_SHA=$(git rev-parse HEAD~1)
```

**2. Select and dispatch the review agent:**

- Use Task tool with `superpowers:code-reviewer-light` for Light Review and fill template at `code-reviewer-light.md`
- Use Task tool with `superpowers:code-reviewer` for Deep Formal Review and fill template at `code-reviewer.md`

**Placeholders:**
- `{WHAT_WAS_IMPLEMENTED}` - What you just built
- `{PLAN_REFERENCE}` - The plan document, pasted requirements, or other source of truth for what it should do
- `{BASE_SHA}` - Starting commit
- `{HEAD_SHA}` - Ending commit
- `{DESCRIPTION}` - Brief summary

**3. Act on feedback:**
- If the reviewer returns feedback that needs evaluation, use `superpowers:receiving-code-review`
- Fix Critical issues immediately
- Fix Important issues before proceeding
- Note Minor issues for later
- Push back if reviewer is wrong (with reasoning)

## Example

```
[All implementation tasks complete]

You: Let me request a final code review before finishing this branch.

BASE_SHA=$(git merge-base HEAD origin/main)  # or the actual base branch / task-series starting SHA
HEAD_SHA=$(git rev-parse HEAD)

[Dispatch superpowers:code-reviewer subagent]
  WHAT_WAS_IMPLEMENTED: Feature implementation across all completed tasks
  PLAN_REFERENCE: docs/superpowers/plans/deployment-plan.md
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: Completed the planned verification, repair, and reporting work

[Subagent returns]:
  Strengths: Clean architecture, real tests
  Issues:
    Important: Missing progress indicators
    Minor: Magic number (100) for reporting interval
  Assessment: Ready to merge with fixes

You: [Fix progress indicators]
[Re-request review if needed, then finish the branch]
```

## Integration with Workflows

**Subagent-Driven Development:**
- Review once after all tasks
- Catch cross-task issues before finishing the branch
- Fix important issues before merge or branch completion

**Executing Plans:**
- Review after all tasks are complete when the work is Full Lane or review thresholds are triggered
- Use `code-reviewer-light` for narrow local reviews and `code-reviewer` for deep formal reviews
- Do not dispatch deep formal review by default for Light Lane work

**Parallel Subagent Execution:**
- Review once after all waves are integrated
- Catch cross-wave integration issues before finishing the branch
- Fix Important issues before merge or branch completion

**Ad-Hoc Development:**
- Review before merge
- Review when stuck

## Red Flags

**Never:**
- Skip a required Full Lane formal review because "it's simple"
- Ignore Critical issues
- Proceed with unfixed Important issues
- Argue with valid technical feedback

**But also never:**
- dispatch review automatically for every small low-risk bugfix
- dispatch review after every task in a multi-task workflow
- treat review as mandatory for isolated local display, formatting, or copy changes

**If reviewer wrong:**
- Push back with technical reasoning
- Show code/tests that prove it works
- Request clarification

See templates at: `requesting-code-review/code-reviewer.md` and `requesting-code-review/code-reviewer-light.md`
