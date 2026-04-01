---
name: executing-plans
description: Use when you have a written implementation plan and want the main agent to execute it directly in the current session step by step
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, and report actual verified status when complete.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** Inline execution means the main agent implements and verifies the work directly in the current session. Do not assume subagents are part of the inline path.

## The Process

### Step 1: Load and Review Plan
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Create TodoWrite and proceed

### Step 2: Execute Tasks

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Mark as completed

### Step 3: Optional Review Gate

After all tasks complete and are verified:
- For Inline Execution, do NOT automatically invoke requesting-code-review
- Request review only when:
  - the user explicitly asks for review
  - the change crosses the review threshold defined by `superpowers:requesting-code-review`
  - persistence, data model, or multi-surface integration risk is involved
- If review is requested or justified by risk, announce: "I'm using the requesting-code-review skill to review the completed implementation."
- Then use `superpowers:requesting-code-review` and address Important or Critical issues before proceeding

### Step 4: Integration Actions

Do NOT invoke finishing-a-development-branch unless the user explicitly asks to:
- commit
- merge
- push
- create a PR

If the user explicitly asks for one of those integration actions:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- Then use `superpowers:finishing-a-development-branch`

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent

## Integration

**Required workflow skills:**
- **superpowers:using-git-worktrees** - REQUIRED: Set up isolated workspace before starting
- **superpowers:writing-plans** - Creates the plan this skill executes
- **superpowers:requesting-code-review** - Optional for inline execution; use when review is requested or risk justifies it
- **superpowers:finishing-a-development-branch** - Use only for explicit integration actions
