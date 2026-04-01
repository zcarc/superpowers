# Skill And Agent Flow Review

Date: 2026-04-01
Project: `timer`
Scope: This document reviews what actually happened in the current session, why it happened, what was excessive for a personal project fast-safe workflow, and how to improve the trigger rules for skills and agents.

## Executive Summary

The current Superpowers setup is optimized more for disciplined team workflows than for fast iteration on a personal project.

In this session, the actual flow became:

`using-superpowers -> systematic-debugging -> brainstorming -> writing-plans -> executing-plans -> test-driven-development -> implementation -> requesting-code-review -> review subagent(s) -> receiving-code-review -> re-review subagent(s) -> finishing-a-development-branch -> verification-before-completion`

That was heavier than necessary for this bugfix.

The biggest mismatches were:

1. `Inline Execution` still caused review subagent dispatch.
2. Review subagents themselves reported invoking additional skills.
3. Small bugfix work went through design approval, plan writing, execution workflow, repeated review, and branch-finishing flow.

For a personal project where speed matters but correctness still matters, a better default is:

`using-superpowers -> systematic-debugging -> test-driven-development -> inline implementation -> verification-before-completion`

Then add planning or review layers only when change size or risk justifies them.

## Actual Session Timeline

This section records what actually happened in this session, in order.

### 1. Main agent startup

Triggered component:
- `using-superpowers`

Why it fired:
- The current root rule says to invoke relevant skills before any response or action.
- `skills-custom/using-superpowers/SKILL.md` makes this mandatory at conversation start.

Assessment:
- Reasonable as a root bootstrap skill.

### 2. Initial bug investigation

Triggered component:
- `systematic-debugging`

Why it fired:
- The user reported incorrect behavior: custom 5-second work item ended up showing `예정 1분` in the review screen.
- This is a bug report and fits the debugging workflow.

Assessment:
- Appropriate.

### 3. Design-mode transition

Triggered component:
- `brainstorming`

Why it fired:
- The user said `설계 진행`.
- The current brainstorming skill hard-gates all modification work behind design and approval.

What happened:
- Project context was explored.
- A scope question was asked.
- 3 approaches were proposed.
- A recommended design was presented.
- The user approved the design.

Assessment:
- Technically consistent with the skill.
- Heavy for a small, well-scoped bugfix.

### 4. Plan generation

Triggered component:
- `writing-plans`

Why it fired:
- `brainstorming` explicitly says the next step after approved design is `writing-plans`.

What happened:
- A detailed plan file was written to:
  - `docs/superpowers/plans/2026-04-01-work-mode-subminute-duration-display-plan.md`
- The plan recommended:
  - `1. Inline Execution`

Assessment:
- Consistent with current skill chain.
- Overkill for this fix in a personal project context.

### 5. Inline execution choice

Triggered component:
- `executing-plans`

Why it fired:
- The plan said `If Inline Execution chosen: REQUIRED SUB-SKILL: Use superpowers:executing-plans`.
- The user selected `1`.

Important clarification:
- In the current design, `Inline Execution` means the main agent performs the implementation tasks directly.
- It does not mean `no subagents at all`.

Assessment:
- This is the key naming mismatch that caused confusion.

### 6. Test-first implementation

Triggered component:
- `test-driven-development`

Why it fired:
- The current TDD skill says it should be used for feature work, bugfixes, refactors, and behavior changes.

What happened:
- Failing tests were added first.
- Failures were verified.
- Minimal implementation was added.
- Tests were re-run.

Assessment:
- Appropriate and valuable.

### 7. Main-agent implementation work

Triggered component:
- No subagent for implementation

What happened:
- The main agent directly edited and verified these areas:
  - `src/renderer/lib/format.ts`
  - `src/renderer/lib/work-mode.ts`
  - `src/renderer/components/work-session-review.tsx`
  - `src/renderer/components/work-mode-panel.tsx`
  - `src/renderer/components/work-task-list.tsx`
  - related tests

Assessment:
- This part matched the user's expectation of inline implementation.

### 8. Final review step after implementation

Triggered component:
- `requesting-code-review`

Why it fired:
- `executing-plans` Step 3 requires final review after tasks complete.
- `requesting-code-review` explicitly says to dispatch a code-reviewer subagent.

What happened:
- First attempt to dispatch `code-reviewer` failed with `ProviderModelNotFoundError`.
- A fallback `general` subagent was then used as a reviewer.

Assessment:
- This is where `inline` still led to subagent usage.
- This behavior was consistent with the current skill chain, but inconsistent with the user's likely expectation of `main-agent-only inline`.

### 9. Main agent received review findings

Triggered component:
- `receiving-code-review`

Why it fired:
- Review feedback arrived from the subagent.
- The receiving-code-review workflow is designed to make the main agent verify review feedback before acting on it.

What happened:
- The main agent checked the findings.
- One review finding was accepted and fixed.
- New regression tests were added.

Assessment:
- Reasonable once a review subagent has been used.
- But this layer only exists because the review subagent was used in the first place.

### 10. Re-review loop

Triggered component:
- additional `general` review subagents

Why it fired:
- After the main agent addressed findings, it requested review again to confirm the fix.

What happened:
- Multiple review subagent sessions were used.
- Their outputs were used to refine the implementation.

Assessment:
- Useful for high-risk work.
- Too heavy for a small UI bugfix in a personal project workflow.

### 11. Branch-finishing workflow

Triggered component:
- `finishing-a-development-branch`

Why it fired:
- `executing-plans` Step 4 requires it after review is complete.

What happened:
- Full `npm test` was run.
- It failed for unrelated `.opencode/node_modules/zod/...` tests.
- The session then reported the real status instead of claiming success.

Assessment:
- The verification discipline was good.
- The branch-finishing workflow itself was unnecessary because the user did not ask to commit, merge, or create a PR.

### 12. Verification before completion

Triggered component:
- `verification-before-completion`

Why it fired:
- Before any final success claim, the verification skill requires fresh evidence.

Assessment:
- Good principle.
- The selected verification command was broader than needed for this project because it picked up unrelated `.opencode` tests.

## Actual Main-Agent Skill Trigger Conditions In This Session

This section explains what triggered each main-agent skill in practice.

| Component | Trigger in this session | Was it appropriate? | Notes |
| --- | --- | --- | --- |
| `using-superpowers` | Conversation start | Yes | Root bootstrap skill |
| `systematic-debugging` | Bug report with incorrect behavior | Yes | Good fit |
| `brainstorming` | User explicitly asked for design progression | Partly | Valid by current rules, but too heavy for a small bugfix |
| `writing-plans` | Required by `brainstorming` after design approval | Mostly no | Useful for larger work, excessive here |
| `executing-plans` | User selected `Inline Execution` from the generated plan | Partly | Inline implementation yes, but later review chain was heavy |
| `test-driven-development` | Bugfix / behavior change before code edits | Yes | Good fit |
| `requesting-code-review` | Required by `executing-plans` Step 3 | Not ideal | Too heavy for this small fix |
| `receiving-code-review` | Review findings came back | Reasonable | Secondary overhead caused by review subagent usage |
| `finishing-a-development-branch` | Required by `executing-plans` Step 4 | No | User did not ask to commit/merge/PR |
| `verification-before-completion` | Before final claim | Yes | Good principle, but verification scope should be narrower |

## Actual Agent And Subagent Trigger Conditions In This Session

### Main agent to subagent

The main agent dispatched subagents only for review, not for implementation.

#### Attempted agent: `code-reviewer`

Trigger condition:
- `requesting-code-review` says to dispatch a code-reviewer subagent.

Actual result:
- Failed with `ProviderModelNotFoundError`.

#### Actual fallback agent: `general`

Trigger condition:
- The main agent wanted to preserve the required final review step after the dedicated review agent failed.

Actual usage:
- First review
- Re-review after fixes
- Final review

Assessment:
- Fallback behavior was pragmatic.
- Multiple review passes were more than a fast-safe personal workflow needed.

## What Happened Inside The Review Subagents

This section is based on direct follow-up questions asked to the review subagent sessions after the fact.

### Review subagent session A

Session reported:
- Invoked `using-superpowers`, `receiving-code-review`, and `verification-before-completion`
- Did not dispatch additional agents or subagents
- Inspected code, reviewed diffs, and ran focused tests

### Review subagent session B

Session reported:
- Invoked `using-superpowers` and `receiving-code-review`
- Did not dispatch additional agents or subagents
- Inspected code and diffs only

### Review subagent session C

Session reported:
- Invoked `using-superpowers`
- Did not dispatch additional agents or subagents
- Only inspected code and returned findings

## Main Agent After Subagent Return

The question here is: when a subagent returned work to the main agent, did the main agent trigger new skills or agents again?

Yes, that happened.

Actual pattern in this session:

1. Main agent dispatched review subagent.
2. Review subagent returned findings.
3. Main agent invoked `receiving-code-review` to evaluate those findings.
4. Main agent made fixes and ran tests.
5. Main agent dispatched another review subagent for confirmation.
6. After final review, main agent invoked `finishing-a-development-branch` and `verification-before-completion`.

So the current system allows:

`main agent -> subagent -> main agent skills -> subagent again -> main agent skills`

This did happen in the current session.

What did not happen:

- No review subagent dispatched additional subagents.
- No review subagent dispatched the `code-reviewer` agent again.
- There was no infinite agent recursion.

## Core Problems Observed

### Problem 1: `Inline Execution` is misleading

Current effective meaning:
- main agent performs implementation directly
- review may still use subagents

Likely user expectation:
- main agent does the work end-to-end without subagent dispatch unless explicitly requested

Impact:
- User confusion
- Workflow appears to violate the user's selected mode

### Problem 2: Small fixes are forced through heavyweight planning

Current path for even a small bugfix can become:
- design
- approval
- plan document
- execution mode choice
- execution workflow
- code review workflow
- branch finishing workflow

Impact:
- Slow iteration
- Increased token use
- More chances for workflow drift than code errors

### Problem 3: Review subagents are not isolated enough

The review subagents reported invoking skills such as:
- `using-superpowers`
- `receiving-code-review`
- `verification-before-completion`

Impact:
- Review workers are not purely review workers
- They can re-enter workflow logic that belongs to the main agent
- This is especially awkward for `receiving-code-review` inside a review subagent

### Problem 4: Branch-finishing flow is too eager

Current issue:
- The system entered `finishing-a-development-branch` even though the user did not ask for commit, merge, or PR creation.

Impact:
- Extra overhead
- More unrelated commands
- More opportunities to hit environment-specific failures

### Problem 5: Verification scope is too broad

Current issue:
- Full `npm test` included unrelated `.opencode` dependency tests.

Impact:
- Noise
- False sense that the current change is blocked by project-external issues

## Recommended Trigger Rules Per Skill And Agent

This section recommends more appropriate trigger conditions for a personal-project fast-safe workflow.

### Keep as default

#### `using-superpowers`

Recommended trigger:
- conversation start only
- do not re-run inside review-only subagents

#### `systematic-debugging`

Recommended trigger:
- bug reports
- failing tests
- unexpected behavior

#### `test-driven-development`

Recommended trigger:
- any behavior change or bugfix that can be tested

#### `verification-before-completion`

Recommended trigger:
- before claiming done
- verify with the narrowest command that meaningfully proves the fix

### Use conditionally

#### `brainstorming`

Recommended trigger:
- unclear requirements
- meaningful product or UX choice
- new feature definition

Do not trigger for:
- clear bugfixes
- copy changes
- isolated display logic fixes with known expected behavior

#### `writing-plans`

Recommended trigger:
- 3 or more meaningful implementation steps
- multiple state transitions or persistence boundaries
- user explicitly asks for a plan

Do not trigger for:
- small bugfixes with obvious scope

#### `executing-plans`

Recommended trigger:
- only when a real plan exists and the work is large enough to benefit from structured execution

Do not trigger for:
- small direct bugfixes

#### `requesting-code-review`

Recommended trigger:
- 5 or more files changed
- persistence, data model, or synchronization logic touched
- user asks for review
- risky refactor

Do not trigger for:
- small UI label or formatting fixes unless there is a real integration risk

### Remove from automatic path

#### `finishing-a-development-branch`

Recommended trigger:
- only if the user explicitly asks to commit, merge, push, or create a PR

Do not trigger for:
- “implementation complete” alone

#### `code-reviewer` or review `general` subagent

Recommended trigger:
- only when review is justified by scope or user request

Do not trigger for:
- every inline execution by default

## Recommended Trigger Rules For Review Subagents

Review subagents should be much stricter than main agents.

### Recommended allowed behavior

- inspect code
- inspect diffs
- inspect tests
- run focused verification if needed
- return findings

### Recommended disallowed behavior

- do not invoke `using-superpowers`
- do not invoke `brainstorming`
- do not invoke `writing-plans`
- do not invoke `executing-plans`
- do not invoke `receiving-code-review`
- do not invoke `finishing-a-development-branch`
- do not dispatch new subagents unless explicitly instructed by the main agent

### Why

This keeps review workers as review workers.
It prevents workflow recursion and reduces token and latency cost.

## Three Recommended Improved Flows

These are ranked from most recommended to least recommended for this project style.

### 1. Recommended: Personal Fast-Safe Inline

Flow:

`using-superpowers -> classify task -> systematic-debugging if bug -> test-driven-development -> inline implementation by main agent -> focused verification -> optional single review if high risk or requested`

When to use:
- clear bugfixes
- small UI or state fixes
- low-risk behavior changes
- personal project iteration

What changes compared to current flow:
- no mandatory brainstorming for clear bugfixes
- no mandatory plan document
- no mandatory review subagent
- no branch-finishing workflow unless user asks

Advantages:
- fastest path
- preserves correctness through debugging, TDD, and verification
- lowest workflow overhead
- matches intuitive meaning of `inline`

Disadvantages:
- less process documentation
- fewer explicit review checkpoints

Result if selected:
- the same kind of bugfix would likely have taken:
  - `using-superpowers`
  - `systematic-debugging`
  - `test-driven-development`
  - inline fix
  - focused test verification
  - done

### 2. Balanced: Personal Inline With One Review Gate

Flow:

`using-superpowers -> classify task -> optional lightweight design note if ambiguity exists -> test-driven-development -> inline implementation -> focused verification -> one review subagent only if change crosses risk threshold -> done`

When to use:
- medium-sized changes
- 4 to 8 files touched
- state transitions, persistence, or coordination logic changed
- still a personal project, but more safety desired

What changes compared to current flow:
- planning is lightweight and can stay in chat
- only one review gate
- no repeated re-review loops unless the first review finds something important
- no branch-finishing skill without explicit integration request

Advantages:
- good balance of speed and safety
- catches integration mistakes
- less overhead than current full workflow

Disadvantages:
- still slower than pure fast-safe inline
- requires a clear threshold for when review is worth it

Result if selected:
- this bugfix probably would have used no design doc and no plan doc
- it might have used one review pass after implementation, but not multiple loops

### 3. Least Recommended Here: Full Team Workflow

Flow:

`using-superpowers -> systematic-debugging -> brainstorming -> writing-plans -> execution mode selection -> executing-plans -> review subagent(s) -> receiving-code-review -> finishing-a-development-branch -> final verification`

When to use:
- large features
- architecture changes
- multi-person work
- work that will be handed off or merged through formal review

What changes compared to current flow:
- essentially the same as what happened in this session

Advantages:
- strongest documentation trail
- strongest process discipline
- clearer checkpoints for teams

Disadvantages:
- highest overhead
- slower feedback cycle
- easiest to feel “workflow-first” instead of “problem-first”
- not a good default for small personal-project bugfixes

Result if selected:
- exactly the kind of heavy path that happened in this session

## Recommended Choice

For this repository and the style you described, the best default is:

### `Personal Fast-Safe Inline`

This gives the best tradeoff between speed and correctness.

Suggested default policy:

1. Always start with `using-superpowers`.
2. If the task is a clear bugfix, use `systematic-debugging`.
3. If the fix changes behavior, use `test-driven-development`.
4. Implement inline in the main agent.
5. Verify with focused tests.
6. Use review subagents only when risk justifies them.
7. Only use branch-finishing workflow when the user explicitly asks for integration actions.

## Concrete Improvement Actions

### Priority 1

Redefine `Inline Execution` to mean:
- no automatic subagent dispatch
- main-agent-only unless the user asks for review or the change crosses a defined risk threshold

### Priority 2

Add a small-bugfix exception to `brainstorming` and `writing-plans`.

Suggested rule:
- if the task is a clearly scoped bugfix with explicit expected behavior and touches a small local area, skip mandatory design and plan-document workflow

### Priority 3

Restrict review subagents so they cannot re-enter general workflow skills.

Suggested rule:
- review subagents may inspect and verify, but may not invoke planning, review-reception, branch-finishing, or additional agent-dispatch flows

### Priority 4

Make `finishing-a-development-branch` opt-in.

Suggested rule:
- only run when the user asks to commit, merge, push, or open a PR

### Priority 5

Narrow verification commands.

Suggested rule:
- prefer project-owned focused verification over repo-wide or environment-wide tests when proving a local bugfix

## Final Assessment

What was true in this session:

1. The implementation itself was inline in the main agent.
2. Review still used subagents because the current `executing-plans -> requesting-code-review` chain requires it.
3. Review subagents reported invoking skills internally.
4. The main agent did trigger more skills after subagent results came back.
5. This was process-correct under the current skill design.
6. It was not the best fit for a fast-safe personal-project workflow.

The most important improvement is to separate these two meanings clearly:

- `inline implementation`
- `main-agent-only execution`

Right now those are not the same thing.
