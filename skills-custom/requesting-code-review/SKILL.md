---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---

# Requesting Code Review

Dispatch superpowers:code-reviewer subagent to catch issues before merge or handoff. The reviewer gets precisely crafted context for evaluation — never your session's history. This keeps the reviewer focused on the work product, not your thought process, and preserves your own context for continued work.

**Core principle:** Review at meaningful checkpoints.

## When to Request Review

**Mandatory:**
- After all tasks in subagent-driven development
- After completing major feature
- Before merge to main

**Optional but valuable:**
- When stuck (fresh perspective)
- Before refactoring (baseline check)
- After fixing complex bug

## How to Request

**1. Get git SHAs:**
```bash
# Default: review the full branch or completed task series
BASE_SHA=$(git merge-base HEAD origin/main)  # or origin/master / actual base branch
HEAD_SHA=$(git rev-parse HEAD)

# Only for a single-commit spot check:
# BASE_SHA=$(git rev-parse HEAD~1)
```

**2. Dispatch code-reviewer subagent:**

Use Task tool with superpowers:code-reviewer type, fill template at `code-reviewer.md`

**Placeholders:**
- `{WHAT_WAS_IMPLEMENTED}` - What you just built
- `{PLAN_OR_REQUIREMENTS}` - What it should do
- `{BASE_SHA}` - Starting commit
- `{HEAD_SHA}` - Ending commit
- `{DESCRIPTION}` - Brief summary

**3. Act on feedback:**
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
  PLAN_OR_REQUIREMENTS: docs/superpowers/plans/deployment-plan.md
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
- Review after a major batch if the change is risky or spans multiple tasks
- Always request review before merge when the completed change set is substantial

**Ad-Hoc Development:**
- Review before merge
- Review when stuck

## Red Flags

**Never:**
- Skip review because "it's simple"
- Ignore Critical issues
- Proceed with unfixed Important issues
- Argue with valid technical feedback

**If reviewer wrong:**
- Push back with technical reasoning
- Show code/tests that prove it works
- Request clarification

See template at: requesting-code-review/code-reviewer.md
