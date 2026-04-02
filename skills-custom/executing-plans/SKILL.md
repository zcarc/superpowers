---
name: executing-plans
description: Use when the selected execution mode is main-agent direct inline execution for a written implementation plan
---

# Executing Plans

## Overview

Execute a written implementation plan directly in the main agent, step by step, with normal verification and explicit handoff only when needed.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Boundary:** This skill applies only after Inline Execution has been selected. If the selected mode is Subagent-Driven or Parallel Subagents, use those skills instead.

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
4. Do any local self-checks needed, but do not dispatch routine per-task formal review in this workflow
5. Mark as completed

### Step 3: Finish According To Lane

After all tasks complete and are verified:

- If this work is Light Lane and no review trigger fired:
  - do a final local self-check
  - summarize the verification evidence
  - proceed without formal review

- If this work is Full Lane, or if review was explicitly requested, or if review thresholds were triggered:
  - announce: "I'm using the requesting-code-review skill to review the completed implementation."
  - then use `superpowers:requesting-code-review` to review the completed integrated result
  - if it returns findings, use `superpowers:receiving-code-review` to evaluate them and address Important or Critical issues before proceeding
  - if it returns no actionable findings, proceed

### Step 4: Optional Integration Handoff

Do NOT invoke `superpowers:finishing-a-development-branch` unless the user explicitly asks for an integration action such as committing, merging, pushing, or creating a PR.

If the user explicitly asks for an integration action:
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
- **superpowers:requesting-code-review** - Use for Full Lane work or when review thresholds are triggered
- **superpowers:receiving-code-review** - Use when a light or deep review returns feedback that needs evaluation
- **superpowers:finishing-a-development-branch** - Use only for explicit integration actions
