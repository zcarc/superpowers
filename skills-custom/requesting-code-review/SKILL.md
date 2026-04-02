---
name: requesting-code-review
description: Use when a completed integrated change set needs formal review before integration or handoff
---

# Requesting Code Review

Dispatch superpowers:code-reviewer subagent to catch issues in the completed integrated result before merge or handoff. The reviewer gets precisely crafted context for evaluation — never your session's history. This keeps the reviewer focused on the work product, not your thought process, and preserves your own context for continued work.

**Core principle:** Review the completed integrated result at meaningful checkpoints, not every task by default.

## When to Request Review

**Mandatory:**
- After all tasks in executing-plans
- After all tasks in subagent-driven development
- After all waves in parallel subagent execution
- After completing major feature
- Before merge to main

**Optional but valuable:**
- When stuck (fresh perspective)
- Before refactoring (baseline check)
- After fixing complex bug

Routine per-task formal review is not the default workflow.

## Review Timing

- Standard workflow timing: request formal review once after all implementation tasks are complete and the integrated result has been verified.
- Review the completed branch or task series after integration — not a routine per-task checkpoint.
- Use `superpowers:receiving-code-review` only if the review returns feedback that needs technical evaluation.

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

## How to Request

**1. Get git SHAs:**
```bash
# Default: review the completed integrated result across the full branch or task series
BASE_SHA=$(git merge-base HEAD origin/main)  # or origin/master / actual base branch
HEAD_SHA=$(git rev-parse HEAD)

# Only for a single-commit spot check:
# BASE_SHA=$(git rev-parse HEAD~1)
```

**2. Dispatch code-reviewer subagent:**

Use Task tool with superpowers:code-reviewer type, fill template at `code-reviewer.md`

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
- Review once after all tasks are complete
- Review the completed integrated result, not each task individually
- Fix Important issues before merge or branch completion

**Parallel Subagent Execution:**
- Review once after all waves are integrated
- Catch cross-wave integration issues before finishing the branch
- Fix Important issues before merge or branch completion

**Ad-Hoc Development:**
- Review before merge
- Review when stuck

## Red Flags

**Never:**
- Skip a workflow's required final formal review because "it's simple"
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

See template at: requesting-code-review/code-reviewer.md
