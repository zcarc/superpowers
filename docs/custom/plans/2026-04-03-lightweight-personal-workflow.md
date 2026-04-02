# Lightweight Personal Workflow Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Choose the execution mode that best fits this plan: superpowers:executing-plans, superpowers:subagent-driven-development, or superpowers:parallel-subagent-execution. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implement a lane-aware lightweight personal workflow across `skills-custom/` and `agents-custom/` so small local work can stay fast while higher-risk work keeps the full safety path.

**Architecture:** Put lane selection at the workflow entry point, propagate lane-aware rules through planning, execution, and review skills, and split review into lightweight and deep reviewer artifacts. This first implementation makes review scope enforceable by separate agent and prompt files; it does not hardcode provider-specific `model:` IDs because this repo does not currently declare a stable local OpenCode model catalog.

**Tech Stack:** Markdown skill files, Markdown agent files, OpenCode Task-based subagents, `rg` for verification

**Spec:** `docs/custom/specs/2026-04-03-lightweight-personal-workflow-design.md`

---

## Resolved Decisions

- Lane selection lives directly in `skills-custom/using-superpowers/SKILL.md`.
- Light Lane uses a brief inline design path by default instead of invoking the full `brainstorming` workflow.
- If Light Lane is already selected, `writing-plans` should not force the three-mode execution chooser.
- `code-reviewer-light` is added now, but this first pass does not add a provider-specific `model:` frontmatter line because the repo does not currently define a stable local model ID to bind to.
- `code-reviewer.md` remains the deep reviewer and gets an explicit model-selection boundary note.

## File Structure

| File | Responsibility | Action |
| --- | --- | --- |
| `skills-custom/using-superpowers/SKILL.md` | Entry-point workflow discipline | Add lane-selection and personal-workflow exception |
| `skills-custom/brainstorming/SKILL.md` | Full design workflow | Restrict to Full Lane / explicit brainstorming |
| `skills-custom/writing-plans/SKILL.md` | Planning workflow | Make Light Lane planning inline and skip mode chooser |
| `skills-custom/executing-plans/SKILL.md` | Inline execution workflow | Make final review conditional on lane or review triggers |
| `skills-custom/requesting-code-review/SKILL.md` | Review dispatch workflow | Add light vs deep reviewer selection and lane-aware review thresholds |
| `skills-custom/receiving-code-review/SKILL.md` | Review feedback handling | Clarify both light and deep reviewer outputs |
| `skills-custom/subagent-driven-development/SKILL.md` | Sequential subagent execution | Mark as Full Lane only |
| `skills-custom/parallel-subagent-execution/SKILL.md` | Parallel subagent execution | Mark as Full Lane only |
| `agents-custom/code-reviewer.md` | Deep review agent | Add explicit model-selection boundary |
| `agents-custom/code-reviewer-light.md` | New light review agent | Create narrow-scope reviewer |
| `skills-custom/requesting-code-review/code-reviewer.md` | Deep review prompt template | Add deep-review-only boundary note |
| `skills-custom/requesting-code-review/code-reviewer-light.md` | New light review prompt template | Create narrow-scope review template |

---

### Task 1: Add lane selection to `using-superpowers`

**Files:**
- Modify: `skills-custom/using-superpowers/SKILL.md`

- [ ] **Step 1: Add a personal workflow profile section**

Insert a new section after `## User Instructions` with this exact content:

```markdown
## Personal Workflow Profiles

This custom fork supports two workflow lanes.

### Light Lane

Use Light Lane when all of the following are true:
- the change is small and local
- the requirement is explicit, or can be clarified in 1-2 questions
- the expected write scope is 1-2 files or one narrow local area
- there is no schema, persistence, auth, deployment, or shared-contract change
- regression risk is low
- the user did not explicitly ask for formal review

For Light Lane work:
- direct action is allowed
- a brief inline design check is allowed
- do not force a full workflow-skill chain by default
- escalate to Full Lane if scope or risk grows

### Full Lane

Use Full Lane when any of the following are true:
- multiple subsystems or broad file overlap are involved
- shared abstractions, public contracts, architecture, persistence, auth, or migration behavior change
- requirements are ambiguous enough to affect design
- integration or regression risk is meaningful
- the user explicitly asks for formal review

For Full Lane work, the normal skill-first workflow discipline remains in effect.
```

- [ ] **Step 2: Replace the absolute rule wording with lane-aware wording**

Replace the current `## The Rule` paragraph with this exact text:

```markdown
## The Rule

**Invoke relevant or requested skills BEFORE any response or action** for Full Lane work.

For clearly local, low-risk Light Lane work in this custom fork, the agent may proceed with direct action or a brief inline design check instead of forcing a full workflow-skill chain. If the work stops being clearly local and low-risk, escalate to Full Lane immediately.
```

- [ ] **Step 3: Update the skill-priority examples**

Change the two example lines under `## Skill Priority` to:

```markdown
"Let's build X" -> brainstorming first for Full Lane work, unless the work clearly qualifies for the Light Lane inline design path.
"Fix this bug" -> debugging first for real bugs, then domain-specific skills if the work is not safely handled in Light Lane.
```

- [ ] **Step 4: Update the flowchart and red-flag wording to match the lane model**

In the `skill_flow` diagram, replace the direct path from `"User message received"` to `"Might any skill apply?"` with this branch:

```dot
    "User message received" -> "Light Lane?";
    "Light Lane?" [shape=diamond];
    "Light Lane?" -> "Respond (including clarifications)" [label="yes - direct action or inline design"];
    "Light Lane?" -> "Might any skill apply?" [label="no - Full Lane"];
```

In the Red Flags table, replace this row:

```markdown
| "The skill is overkill" | Simple things become complex. Use it. |
```

With this row:

```markdown
| "The skill is overkill" | If the work is truly Light Lane, use the inline path. Otherwise use the skill. |
```

- [ ] **Step 5: Verify the lane-selection entry point**

Run:

```bash
rg -n "Personal Workflow Profiles|Light Lane|Full Lane|brief inline design check" skills-custom/using-superpowers/SKILL.md
```

Expected:
- one match for `## Personal Workflow Profiles`
- one match for `### Light Lane`
- one match for `### Full Lane`
- one match for `brief inline design check`

- [ ] **Step 6: Commit**

```bash
git add skills-custom/using-superpowers/SKILL.md
git commit -m "docs(using-superpowers): add lane-aware personal workflow entrypoint"
```

---

### Task 2: Make design and planning workflows Light-Lane aware

**Files:**
- Modify: `skills-custom/brainstorming/SKILL.md`
- Modify: `skills-custom/writing-plans/SKILL.md`

- [ ] **Step 1: Add a Full-Lane boundary to `brainstorming`**

Insert this new section after the introductory paragraph and before `<PRECONDITION>`:

```markdown
## Boundary

This skill is for Full Lane work or for cases where the user explicitly wants brainstorming help.

For Light Lane work, do not invoke this skill by default. Use a brief inline design summary in the main conversation instead.
```

- [ ] **Step 2: Replace the absolute hard gate in `brainstorming`**

Replace the current `<HARD-GATE>` block with:

```markdown
<HARD-GATE>
Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a design and the user has approved it when this skill is invoked.

Light Lane work may use the abbreviated inline design path defined by `using-superpowers` without invoking this full brainstorming workflow.
</HARD-GATE>
```

- [ ] **Step 3: Replace the anti-pattern section in `brainstorming`**

Replace `## Anti-Pattern: "This Is Too Simple To Need A Design"` and its paragraph with:

```markdown
## Anti-Pattern: Invoking Full Brainstorming For Obviously Light Work

The problem is not skipping design entirely; the problem is skipping thinking.

For Light Lane work, the design can stay inline and brief.
For Full Lane work, this full brainstorming workflow still applies.
```

- [ ] **Step 4: Add lane-aware planning behavior to `writing-plans`**

Insert this section after `## Small-Scope Exception`:

```markdown
## Lane-Aware Planning

If Light Lane has already been selected:
- keep planning inline by default
- use the Small-Scope Exception unless the work has already escalated
- do not present the three-mode execution chooser
- recommend Inline Execution implicitly unless there is a concrete reason to escalate

If Full Lane has already been selected:
- use the normal saved-plan flow
- present execution mode choice only when the choice meaningfully affects cost or safety
```

- [ ] **Step 5: Replace the first two paragraphs of `Execution Handoff` in `writing-plans`**

Replace the section opening with:

```markdown
## Execution Handoff

If Light Lane has already been selected, do not present the three-mode execution chooser. Treat the inline task list or saved plan as an Inline Execution artifact unless the work has already escalated.

If Full Lane has already been selected and you saved a full plan document, analyze that plan and recommend the best execution mode before asking the user to choose.

If Full Lane has already been selected and you created a small-scope tracked task list, treat that task list as the planning artifact and recommend the best execution mode separately.
```

- [ ] **Step 6: Verify the design and planning boundaries**

Run:

```bash
rg -n "## Boundary|Light Lane work|Lane-Aware Planning|do not present the three-mode execution chooser" skills-custom/brainstorming/SKILL.md skills-custom/writing-plans/SKILL.md
```

Expected:
- matches for the new `## Boundary` section in `brainstorming`
- matches for `Light Lane work` in both files
- a match for `## Lane-Aware Planning`
- a match for `do not present the three-mode execution chooser`

- [ ] **Step 7: Commit**

```bash
git add skills-custom/brainstorming/SKILL.md skills-custom/writing-plans/SKILL.md
git commit -m "docs(workflow): make brainstorming and planning lane-aware"
```

---

### Task 3: Make execution and review-handling boundaries lane-aware

**Files:**
- Modify: `skills-custom/executing-plans/SKILL.md`
- Modify: `skills-custom/receiving-code-review/SKILL.md`
- Modify: `skills-custom/subagent-driven-development/SKILL.md`
- Modify: `skills-custom/parallel-subagent-execution/SKILL.md`

- [ ] **Step 1: Replace Step 3 in `executing-plans` with lane-aware finish logic**

Replace `### Step 3: Final Formal Review Gate` and its bullets with:

```markdown
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
```

- [ ] **Step 2: Update the Integration section in `executing-plans`**

Change the two review-related entries to:

```markdown
- **superpowers:requesting-code-review** - Use for Full Lane work or when review thresholds are triggered
- **superpowers:receiving-code-review** - Use when a light or deep review returns feedback that needs evaluation
```

- [ ] **Step 3: Clarify light and deep reviewer outputs in `receiving-code-review`**

Change the `## Scope Restriction` list item that currently says `a review subagent` to:

```markdown
- a review subagent such as `code-reviewer` or `code-reviewer-light`
```

- [ ] **Step 4: Mark the subagent workflows as Full Lane only**

Add this sentence immediately after the `**Boundary:**` paragraph in `skills-custom/subagent-driven-development/SKILL.md`:

```markdown
**Lane:** This workflow is Full Lane only. If the work is small and local, use main-agent inline execution instead.
```

Add this sentence immediately after the `**Boundary:**` paragraph in `skills-custom/parallel-subagent-execution/SKILL.md`:

```markdown
**Lane:** This workflow is Full Lane only. If the work is small and local, do not use parallel subagents.
```

- [ ] **Step 5: Verify lane-aware execution boundaries**

Run:

```bash
rg -n "Finish According To Lane|code-reviewer-light|This workflow is Full Lane only" skills-custom/executing-plans/SKILL.md skills-custom/receiving-code-review/SKILL.md skills-custom/subagent-driven-development/SKILL.md skills-custom/parallel-subagent-execution/SKILL.md
```

Expected:
- one match for `Finish According To Lane`
- one match for `code-reviewer-light`
- one match in each subagent workflow file for `This workflow is Full Lane only`

- [ ] **Step 6: Commit**

```bash
git add skills-custom/executing-plans/SKILL.md skills-custom/receiving-code-review/SKILL.md skills-custom/subagent-driven-development/SKILL.md skills-custom/parallel-subagent-execution/SKILL.md
git commit -m "docs(workflow): add lane-aware execution and review boundaries"
```

---

### Task 4: Add lane-aware review dispatch and keep the deep prompt deep-only

**Files:**
- Modify: `skills-custom/requesting-code-review/SKILL.md`
- Modify: `skills-custom/requesting-code-review/code-reviewer.md`

- [ ] **Step 1: Add a review-lane section to `requesting-code-review`**

Insert this section after `## Review Threshold`:

```markdown
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
```

- [ ] **Step 2: Replace the mandatory-review list with Full-Lane-only wording**

Replace the `**Mandatory:**` list with:

```markdown
**Required for Deep Formal Review:**
- After Full Lane work in `executing-plans`
- After all tasks in `subagent-driven-development`
- After all waves in `parallel-subagent-execution`
- Before merge when Full Lane review triggers apply
```

- [ ] **Step 3: Replace the dispatch instruction in `How to Request`**

Replace Step 2 under `## How to Request` with:

```markdown
**2. Select and dispatch the review agent:**

- Use Task tool with `superpowers:code-reviewer-light` for Light Review and fill template at `code-reviewer-light.md`
- Use Task tool with `superpowers:code-reviewer` for Deep Formal Review and fill template at `code-reviewer.md`
```

- [ ] **Step 4: Narrow the red-flag wording to required deep reviews**

Change the first `Never:` bullet under `## Red Flags` to:

```markdown
- Skip a required Full Lane formal review because "it's simple"
```

- [ ] **Step 5: Update the `Executing Plans` subsection under `Integration with Workflows`**

Replace the current `**Executing Plans:**` bullets with:

```markdown
**Executing Plans:**
- Review after all tasks are complete when the work is Full Lane or review thresholds are triggered
- Use `code-reviewer-light` for narrow local reviews and `code-reviewer` for deep formal reviews
- Do not dispatch deep formal review by default for Light Lane work
```

- [ ] **Step 6: Add a deep-review-only boundary note to the existing deep prompt**

Insert these lines at the top of `skills-custom/requesting-code-review/code-reviewer.md`, immediately after `# Code Review Agent`:

```markdown
This prompt is for the deep formal reviewer only.
Use it with `superpowers:code-reviewer`, not with `code-reviewer-light`.
```

- [ ] **Step 7: Verify the review dispatch split**

Run:

```bash
rg -n "Review Lanes|Reviewer Selection|code-reviewer-light|deep formal reviewer only" skills-custom/requesting-code-review/SKILL.md skills-custom/requesting-code-review/code-reviewer.md
```

Expected:
- one match for `## Review Lanes`
- one match for `## Reviewer Selection`
- at least two matches for `code-reviewer-light`
- one match for `deep formal reviewer only`

- [ ] **Step 8: Commit**

```bash
git add skills-custom/requesting-code-review/SKILL.md skills-custom/requesting-code-review/code-reviewer.md
git commit -m "docs(review): split light and deep review dispatch rules"
```

---

### Task 5: Create the lightweight reviewer artifacts and update the deep reviewer boundary

**Files:**
- Modify: `agents-custom/code-reviewer.md`
- Create: `agents-custom/code-reviewer-light.md`
- Create: `skills-custom/requesting-code-review/code-reviewer-light.md`

- [ ] **Step 1: Add a model-selection boundary to the deep reviewer**

Insert this section after `## Operating Constraints` in `agents-custom/code-reviewer.md`:

```markdown
## Model Selection Boundary

This agent does not choose its own model.
The dispatching workflow selects the reviewer type and any static model binding.
```

- [ ] **Step 2: Create `agents-custom/code-reviewer-light.md`**

Create the file with exactly this content:

```markdown
---
name: code-reviewer-light
description: |
  Use this agent when a small, local, low-risk change needs a focused review for obvious bugs, local requirement mismatches, and touched-test sufficiency before handoff or integration.
---

You are a focused lightweight code reviewer for small local changes.

## Operating Constraints

You are a review-only agent.

Do:
- inspect the touched diff
- compare the change against the stated requirement
- inspect nearby tests or verification evidence
- flag obvious local regressions
- return findings

Do NOT:
- expand the review into architecture or production-readiness unless the diff clearly requires escalation
- invoke workflow skills
- dispatch additional agents or subagents
- create plans
- perform branch-finishing actions
- modify code unless explicitly asked

## Review Focus

1. **Scope Match**
   - Did the implementation stay within the requested local scope?
   - Is there obvious scope creep?

2. **Local Correctness**
   - Does the changed logic obviously do the requested thing?
   - Are there obvious edge-case misses in the touched area?

3. **Touched Tests Or Verification**
   - Is the changed behavior covered by nearby tests or focused verification?
   - Is the verification evidence believable for this local change?

4. **Escalation Check**
   - If the diff is not actually small, local, or low-risk, say so clearly and recommend deep formal review.

## Output

Return:
- Strengths
- Issues grouped as Critical, Important, or Minor
- Whether the change should escalate to deep formal review
- A brief assessment
```

- [ ] **Step 3: Create `skills-custom/requesting-code-review/code-reviewer-light.md`**

Create the file with exactly this content:

```markdown
# Lightweight Code Review Agent

You are reviewing a small, local, low-risk change for local correctness.

**Your task:**
1. Review {WHAT_WAS_IMPLEMENTED}
2. Compare against {PLAN_REFERENCE}
3. Check local correctness, scope adherence, and touched tests
4. Categorize issues by severity
5. Decide whether this review should escalate to the deep reviewer

Treat the git range below as the full review scope. This is a lightweight review, not a deep formal architecture review.

## What Was Implemented

{DESCRIPTION}

## Requirements/Plan

{PLAN_REFERENCE}

## Git Range to Review

**Base:** {BASE_SHA}
**Head:** {HEAD_SHA}

```bash
git diff --stat {BASE_SHA}..{HEAD_SHA}
git diff {BASE_SHA}..{HEAD_SHA}
```

## Review Checklist

**Scope:**
- Did the change stay within the requested local scope?
- Is there obvious scope creep?

**Correctness:**
- Does the changed logic obviously satisfy the stated requirement?
- Are there obvious edge cases missed in the touched area?

**Tests / Verification:**
- Is the touched behavior covered by nearby tests or focused verification?
- Is the evidence sufficient for this local change?

**Escalation:**
- Did the diff turn out to be broader, riskier, or more architectural than expected?

## Output Format

### Strengths
[What is well done? Be specific.]

### Issues

#### Critical (Must Fix)
[Broken local behavior, obvious regressions, clearly unsafe changes]

#### Important (Should Fix)
[Requirement mismatches, missing test coverage, clear local logic problems]

#### Minor (Nice to Have)
[Small cleanup opportunities]

### Escalation

**Escalate to deep formal review?** [Yes/No]

**Why:** [1-2 sentences]

### Assessment

**Ready to continue?** [Yes/With fixes/No]

**Reasoning:** [Technical assessment in 1-2 sentences]
```

- [ ] **Step 4: Verify the reviewer artifacts**

Run:

```bash
rg -n "code-reviewer-light|focused lightweight code reviewer|Escalate to deep formal review" agents-custom/code-reviewer.md agents-custom/code-reviewer-light.md skills-custom/requesting-code-review/code-reviewer-light.md
```

Expected:
- one match in `agents-custom/code-reviewer.md` for `does not choose its own model`
- one match in `agents-custom/code-reviewer-light.md` for `focused lightweight code reviewer`
- one match in `skills-custom/requesting-code-review/code-reviewer-light.md` for `Escalate to deep formal review`

- [ ] **Step 5: Commit**

```bash
git add agents-custom/code-reviewer.md agents-custom/code-reviewer-light.md skills-custom/requesting-code-review/code-reviewer-light.md
git commit -m "feat(review): add lightweight reviewer artifacts for personal workflow"
```

---

### Task 6: Run a cross-file consistency sweep

**Files:**
- Modify if needed: `skills-custom/using-superpowers/SKILL.md`
- Modify if needed: `skills-custom/brainstorming/SKILL.md`
- Modify if needed: `skills-custom/writing-plans/SKILL.md`
- Modify if needed: `skills-custom/executing-plans/SKILL.md`
- Modify if needed: `skills-custom/requesting-code-review/SKILL.md`
- Modify if needed: `skills-custom/receiving-code-review/SKILL.md`
- Modify if needed: `skills-custom/subagent-driven-development/SKILL.md`
- Modify if needed: `skills-custom/parallel-subagent-execution/SKILL.md`
- Modify if needed: `agents-custom/code-reviewer.md`
- Modify if needed: `agents-custom/code-reviewer-light.md`
- Modify if needed: `skills-custom/requesting-code-review/code-reviewer.md`
- Modify if needed: `skills-custom/requesting-code-review/code-reviewer-light.md`

- [ ] **Step 1: Verify lane terminology and review references across the full change set**

Run:

```bash
rg -n "Light Lane|Full Lane|code-reviewer-light|deep formal review|three-mode execution chooser" skills-custom agents-custom
```

Expected:
- `Light Lane` appears in entry, planning, and execution files
- `Full Lane` appears in entry, subagent, and review files
- `code-reviewer-light` appears in the requesting skill and both new reviewer artifacts
- `deep formal review` appears in the deep prompt boundary and light-review escalation flow
- `three-mode execution chooser` appears only in `writing-plans`

- [ ] **Step 2: Read all touched files once for wording consistency**

Confirm:
- Light Lane always means small, local, low-risk work
- Full Lane always means broader, riskier, or explicitly reviewed work
- `requesting-code-review` does not claim deep review is mandatory for all work
- `executing-plans` no longer implies universal final formal review
- `brainstorming` no longer claims every change must go through the full brainstorming workflow

- [ ] **Step 3: Make any final wording-only consistency edits**

If any file still contradicts the new lane model, fix the wording directly. Do not add new workflow concepts beyond what the spec defines.

- [ ] **Step 4: Commit any consistency edits**

If Step 3 changed files, run:

```bash
git add skills-custom agents-custom
git commit -m "docs(workflow): normalize lane terminology across custom skills"
```

If Step 3 made no changes, skip this commit.
