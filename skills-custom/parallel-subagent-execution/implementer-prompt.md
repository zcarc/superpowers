# Parallel Implementer Subagent Prompt Template

Use this template when dispatching an implementer subagent for a task in a parallel wave.

```
Task tool (general-purpose):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N: [task name] as part of a parallel execution wave.

    ## Task Description

    [FULL TEXT of task from plan - paste it here, don't make subagent read file]

    ## Context

    [Scene-setting: where this fits, dependencies already satisfied, architectural context]

    ## File Ownership

    You own only these files for this task:
    - [exact/path/one]
    - [exact/path/two]

    Other subagents may be working in parallel on other tasks.

    Do not modify files outside your assigned scope unless you stop and report
    BLOCKED or NEEDS_CONTEXT. Do not revert changes outside your scope.

    ## Assigned Branch and Worktree

    Work only in this assigned branch and worktree:
    - Branch: [branch name]
    - Worktree: [worktree path]

    Do not switch to another task's branch or worktree.
    Do not integrate other task branches yourself.

    ## Interface Contract

    [Any shared interface, type, route, component API, schema, or agreed contract]

    If the contract seems wrong or incomplete, stop and ask before proceeding.

    ## Before You Begin

    If anything about the requirements, ownership, tests, or interface contract is
    unclear, ask now. Do not guess.

    ## Your Job

    Once you're clear on requirements:
    1. Implement exactly what the task specifies
    2. Write tests (following TDD if task says to)
    3. Verify implementation works
    4. Commit your work
    5. Self-review
    6. Report back

    Work from: [directory]

    ## Self-Review

    Before reporting back, check:
    - Did I stay within my assigned file scope?
    - Did I stay within my assigned branch and worktree?
    - Did I fully implement the task?
    - Do my tests verify real behavior?
    - Did I introduce assumptions about another task's unfinished work?

    Fix anything you can before reporting back.

    ## Report Format

    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - What you implemented
    - What you tested and test results
    - Files changed
    - Any ownership or integration concerns
    - Self-review findings
```
