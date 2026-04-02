---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for the codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about the toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

## Small-Scope Exception

If the approved design is for a small, local change with clear files and clear acceptance criteria:
- do not require a full saved plan document
- provide a short lightweight checklist in the conversation instead

Use a saved plan document when:
- multiple subsystems are involved
- task sequencing matters
- handoff value is meaningful
- the user explicitly wants a plan document

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Choose the execution mode that best fits this plan: superpowers:executing-plans, superpowers:subagent-driven-development, or superpowers:parallel-subagent-execution. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the code — the engineer may be reading tasks out of order)
- Steps that describe what to do without showing how (code blocks required for code steps)
- References to types, functions, or methods not defined in any task

## Remember
- Exact file paths always
- Complete code in every step — if a step changes code, show the code
- Exact commands with expected output
- DRY, YAGNI, TDD, frequent commits

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task that implements it? List any gaps.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Review Terms

- **Implementer self-review** - The task owner checks their own work before handoff. This is a local quality check, not a formal code review.
- **Integration verification** - Tests or checks run after a task or wave is integrated to confirm the combined result still works. This is not a formal code review.
- **Formal review** - Explicit use of `superpowers:requesting-code-review` against the completed integrated result. Default timing: once after all implementation tasks are complete and integration verification is done, not after every task.
- `superpowers:receiving-code-review` applies only if a formal review actually returns feedback that needs evaluation.

## Execution Handoff

If you saved a full plan document, analyze that plan and recommend the best execution mode before asking the user to choose.

If you wrote a small-scope checklist in the conversation instead, treat that checklist as the planning artifact and recommend the best execution mode separately. The checklist format does not by itself select Inline Execution, though small-scope work will usually fit Inline Execution unless task isolation or parallelism clearly provides more value.

Evaluate:
- Task dependencies
- File overlap between tasks
- Integration risk
- Interface ambiguity
- Expected speedup from parallel execution
- Need for close controller oversight

Then present the recommendation in this format:

**Recommended execution mode:** [exact mode name]

**Why this is the best fit for this plan:**
- [reason 1]
- [reason 2]
- [reason 3]

**Why the other available modes are not the best fit:**
- Add one bullet for every execution mode listed below except the recommended mode.
- Cover each non-selected mode exactly once.
- Format: `- **[exact mode name]** - [why not chosen]`

**Execution modes:**

- **Inline Execution**
  - Execute tasks directly in this session, following the plan task-by-task, then run one final formal review after all tasks are complete and verified
  - Best for small plans, tightly coupled work, or situations where the controller should do the implementation directly
  - Required execution skill: `superpowers:executing-plans`

- **Subagent-Driven**
  - Dispatch one fresh subagent per task, execute tasks sequentially, have each implementer self-review, then run one final formal review after all tasks are complete
  - Best default for multi-task plans where task isolation helps but parallel execution would add risk
  - Required execution skill: `superpowers:subagent-driven-development`

- **Parallel Subagents**
  - Partition the plan into independent, non-conflicting waves, run one fresh subagent per task within each wave, have each implementer self-review, verify each integrated wave, then run one final formal review after all waves are integrated
  - Best for plans with clear ownership, low coupling, and meaningful speedup from parallel work
  - Required execution skill: `superpowers:parallel-subagent-execution`

**Which execution mode do you want to use?**

After the user chooses a mode:
- Load the required execution skill for that mode.
- Follow that mode's workflow exactly.

Mode-specific reminders:

- **Inline Execution**
  - Direct execution in this session, following the plan task-by-task
  - Formal review runs once after all tasks are complete

- **Subagent-Driven**
  - Fresh subagent per task + implementer self-review per task
  - Formal review runs once after all tasks are complete

- **Parallel Subagents**
  - Partition the plan into dependency-safe waves
  - Assign explicit file ownership per task
  - Verify each integrated wave before moving on
  - Formal review runs once after all waves are integrated
