# Lightweight Personal Workflow Design

**Date:** 2026-04-03
**Status:** Draft
**Scope:** `skills-custom/`, `agents-custom/`, `docs/custom/`

## Goal

Create a lightweight workflow profile for the personal-use custom fork that preserves strong guardrails for risky work while removing unnecessary overhead for small local changes.

This design should:
- keep high-signal safety checks for ambiguous, multi-file, or high-risk work
- reduce mandatory design and review overhead for small local work
- make formal review and reviewer model selection risk-based instead of universal
- preserve compatibility with the current `skills-custom/` and `agents-custom/` structure

## Context

Upstream Superpowers is optimized for broad public use, strong agent discipline, and contribution-quality workflow control. This fork serves a different purpose: personal-project work in OpenCode, usually with a single human owner who already controls requirements, implementation, and integration decisions.

The current custom fork already improved several heavy upstream defaults:
- routine per-task formal review was removed
- branch-finishing behavior is limited to explicit integration requests
- `writing-plans` already has a small-scope exception

But the workflow still behaves as if most work needs the full safety envelope. That makes sense for upstream and for high-risk project work, but it is too expensive as the default posture for small personal changes.

## Problems With The Current Custom Workflow

- `using-superpowers` and `brainstorming` still strongly bias almost every modification toward full design gating.
- `writing-plans` allows a small-scope exception, but downstream execution skills still tend to route work back into a final formal review.
- `requesting-code-review` and the current `code-reviewer` remain production-readiness oriented even for simple local changes.
- model selection guidance exists in workflow prose, but reviewer model choice is not encoded where OpenCode can actually enforce it.
- tiny work still asks the human to choose between multiple execution modes too often.

## Design Principles

1. Optimize for solo development without removing high-value safety checks.
2. Make "lightweight" mean lower overhead by default, not lower quality standards.
3. Separate local verification from formal review.
4. Scale planning depth, review depth, and agent usage with actual risk.
5. Prefer the smallest correct delta from the current custom fork.
6. Put runtime-enforceable model choices in dispatch logic or agent config, not only in prompt prose.
7. Allow escalation: work may start in the lightweight path and move into the full path if scope expands or risk increases.

## Workflow Profile Overview

This design introduces two lanes inside the custom workflow profile rather than two completely separate systems:

- **Light Lane** - default for small, explicit, low-risk changes
- **Full Lane** - required for multi-file, ambiguous, or high-risk work

The controller chooses a lane early and may escalate later. The important change is that lightweight work no longer pays the same mandatory cost as full-feature or integration-heavy work.

## Two-Lane Model

### Light Lane

Use the Light Lane when all of the following are true:
- the scope is small and local
- the requirement is explicit, or can be clarified in 1-2 questions
- the expected write scope is 1-2 files or one narrowly bounded local area
- there is no schema, persistence, auth, deployment, or shared-contract change
- regression risk is low
- the user did not explicitly ask for a formal review

Default Light Lane workflow:
1. Brief inline design or assumption check in the conversation
2. Inline task list or direct execution; no saved plan document by default
3. Main-agent implementation by default
4. Focused verification for the touched behavior
5. No formal review by default
6. Branch-finishing workflow only on explicit integration request

Escalate out of the Light Lane when:
- scope expands beyond 2-3 files
- a shared abstraction or public contract must change
- tests reveal non-local impact
- the user explicitly asks for review
- the agent is no longer confident that the work is truly low-risk

### Full Lane

Use the Full Lane when any of the following are true:
- multiple subsystems or broad file overlap are involved
- persistence, schema, storage, auth, or migration behavior changes
- shared abstractions, architecture, or reusable contracts change
- requirements are ambiguous enough to affect design
- there is meaningful integration or regression risk
- the user explicitly asks for formal review
- the work is intended to meet commit, merge, or PR-quality expectations

Default Full Lane workflow:
1. Design-first workflow remains available
2. Full plan or structured task artifact
3. Explicit execution mode choice when it meaningfully affects cost or safety
4. Inline, sequential subagent, or parallel subagent execution based on plan shape
5. Final formal review of the integrated result
6. Review feedback evaluation through `receiving-code-review`
7. Branch-finishing workflow only on explicit integration request

## Proposed Lane Selection Matrix

| Change signal | Default lane |
| --- | --- |
| 1-2 files, isolated local logic, copy, styling, or narrow bug fix | Light |
| 2-3 files in one module, clear requirement, low regression risk | Light, escalate if risk appears |
| Shared abstraction, public API, or reusable contract changes | Full |
| Schema, persistence, storage, auth, or migration changes | Full |
| Large refactor or unclear ownership across files | Full |
| User explicitly asks for review | Full or lightweight review depending on scope |
| User explicitly asks for commit, merge, push, or PR | Keep current lane, but use integration workflow at finish |

When uncertain, prefer the Full Lane.

## Review Policy

Formal review is no longer a universal end-of-work step.

- In the Light Lane, "done" means focused verification plus a local self-check unless review is requested or risk grows.
- In the Full Lane, formal review remains the default finishing gate.
- `receiving-code-review` triggers only when actual review findings exist.
- Formal review scope stays the completed integrated result, not routine per-task checkpoints.

This keeps the important distinction intact:
- **local self-check** is for the task owner
- **formal review** is an explicit workflow event for higher-risk work

## Model Selection Policy

OpenCode can enforce reviewer model choice through agent configuration or through dispatching different agents. A reviewer prompt cannot reliably choose its own runtime model.

This design therefore treats model selection as a dispatch concern, not as an instruction inside the reviewer body.

Proposed reviewer split:
- **`code-reviewer-light`** - fast or standard model, narrow-scope review
- **`code-reviewer`** - most capable available model, full formal review

Reviewer responsibilities:
- **`code-reviewer-light`** checks obvious bugs, local requirement mismatches, touched-test sufficiency, and high-signal local regressions.
- **`code-reviewer`** checks architecture, integration risk, broader correctness, and production-readiness concerns where they are relevant.

Implementation subagents should keep the existing model-selection guidance, but the lightweight profile should bias toward main-agent inline execution before spawning subagents.

## Changes By File

### 1. `skills-custom/using-superpowers/SKILL.md`

Current behavior:
- applies strong universal pressure to load skills before any action
- treats skill usage as effectively mandatory for almost all work

Design change:
- add a personal-workflow exception for clearly local, low-risk changes
- allow direct action or a brief inline design for Light Lane work
- keep the current strong skill discipline for Full Lane work

Why:
- this file currently determines the default cost of almost every task

### 2. `skills-custom/brainstorming/SKILL.md`

Current behavior:
- all creative work requires full design gating regardless of perceived simplicity

Design change:
- keep the full brainstorming workflow for Full Lane work
- allow a short inline design response for Light Lane work instead of a formal brainstorm loop
- keep the clarification discipline when requirements are still unclear

Why:
- personal-project work still benefits from design thinking, but not every local change needs a heavyweight design ceremony

### 3. `skills-custom/writing-plans/SKILL.md`

Current behavior:
- already has a small-scope exception
- still presents three execution modes as a frequent decision point

Design change:
- strengthen the small-scope exception as the default for Light Lane work
- treat inline task lists as the normal planning artifact for Light Lane work
- reserve multi-mode execution choice for plans where the choice meaningfully affects cost or safety

Why:
- small local work should not require a saved plan document or a workflow-selection conversation by default

### 4. `skills-custom/executing-plans/SKILL.md`

Current behavior:
- after tasks complete, a final formal review gate is the default

Design change:
- make the final formal review gate conditional on Full Lane or explicit review triggers
- for Light Lane, completion requires focused verification and a local self-check, not mandatory formal review
- preserve explicit integration handoff behavior

Why:
- this is the main place where the current custom workflow becomes heavy again after earlier simplification

### 5. `skills-custom/requesting-code-review/SKILL.md`

Current behavior:
- improved over upstream, but still acts as the default final review entry point for workflow completion

Design change:
- treat formal review as required only for Full Lane work or explicit review thresholds
- add reviewer selection guidance based on risk and review scope
- route small scoped reviews to a lightweight reviewer when review is useful but a deep formal review is unnecessary

Why:
- review should scale with risk, not with the mere fact that work is complete

### 6. `skills-custom/receiving-code-review/SKILL.md`

Current behavior:
- already scoped fairly well

Design change:
- keep the current technical-evaluation behavior
- clarify that it applies to both light and full reviewer outputs when review feedback actually exists

Why:
- this file is not the main source of overhead and mostly needs boundary consistency, not redesign

### 7. `skills-custom/subagent-driven-development/SKILL.md`

Current behavior:
- one final formal review after all tasks, plus model-selection guidance

Design change:
- explicitly mark this workflow as Full Lane by default
- keep final formal review here
- keep model-selection guidance for implementers

Why:
- if the controller is already paying the cost of sequential subagents, the work is usually beyond the lightweight path

### 8. `skills-custom/parallel-subagent-execution/SKILL.md`

Current behavior:
- optimized for safe parallel execution with formal review after integrated waves

Design change:
- treat this as Full Lane only
- do not create a lightweight branch inside this workflow

Why:
- parallel orchestration is inherently not the lightweight profile

### 9. `agents-custom/code-reviewer.md`

Current behavior:
- one heavy reviewer with architecture and production-readiness expectations

Design change:
- keep this agent as the deep reviewer
- explicitly state that model capability is selected by the dispatcher or static agent config, not by the reviewer itself

Why:
- the existing reviewer is still useful, but it should stop being the implied default for all review requests

### 10. New `agents-custom/code-reviewer-light.md`

Design addition:
- add a lightweight review-only agent for narrow-scope local reviews
- give it a narrower charter: requirement match, obvious bugs, touched tests, and high-signal local risks
- assign a cheaper or standard model statically in agent frontmatter if the local OpenCode setup supports that model

Why:
- OpenCode can enforce model choice at the agent configuration level; prompt prose alone cannot make a single agent dynamically change models

### 11. `skills-custom/requesting-code-review/code-reviewer.md`

Current behavior:
- formal review prompt assumes integrated production-readiness review

Design change:
- keep this prompt aligned with the deep reviewer only
- do not repurpose it for lightweight reviews

Why:
- mixing light and deep review instructions into one prompt will blur severity standards and reviewer expectations

### 12. New `skills-custom/requesting-code-review/code-reviewer-light.md`

Design addition:
- add a separate prompt template for lightweight review requests
- narrow the checklist to local correctness, narrow scope adherence, touched-test sufficiency, and obvious regressions

Why:
- model cost and prompt scope should move together

## What Does Not Change

- `systematic-debugging`, `test-driven-development`, `verification-before-completion`, and `using-git-worktrees` remain in place.
- explicit commit, merge, push, or PR requests still use `finishing-a-development-branch`.
- high-risk work still gets design, planning, verification, and formal review.
- the custom fork still values evidence and focused verification before claiming success.

## Non-Goals

- remove all process from the custom fork
- make every task bypass skills
- weaken TDD or debugging discipline for real bugs and regressions
- collapse all review into casual self-approval
- redesign unrelated upstream skills

## Risks And Tradeoffs

- **Risk:** Light Lane may under-review changes that looked small but were not.
  **Mitigation:** use narrow eligibility and explicit escalation triggers.

- **Risk:** Two lanes introduce a judgment call.
  **Mitigation:** prefer the Full Lane when uncertain.

- **Risk:** splitting reviewers adds another agent and prompt template to maintain.
  **Mitigation:** keep the lightweight reviewer prompt narrow and stable.

- **Risk:** users may overuse the Light Lane because it is faster.
  **Mitigation:** encode hard Full Lane triggers around persistence, architecture, auth, shared contracts, and explicit review requests.

## Open Questions

- Should `using-superpowers` contain the lane-selection rule directly, or should it point to a lightweight-profile reference section?
- Should Light Lane still invoke `brainstorming` for any behavior change, but in abbreviated form?
- Should `code-reviewer-light` use a cheap model or a standard model by default on this machine?
- Should the execution-mode chooser in `writing-plans` be skipped entirely when Light Lane is already selected?
