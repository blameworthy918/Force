The Basic Workflow

    brainstorming - Activates before writing code. Refines rough ideas through questions, explores alternatives, presents design in sections for validation. Saves design document.

    using-git-worktrees - Activates after design approval. Creates isolated workspace on new branch, runs project setup, verifies clean test baseline.

    writing-plans - Activates with approved design. Breaks work into bite-sized tasks (2-5 minutes each). Every task has exact file paths, complete code, verification steps.

    subagent-driven-development, executing-plans, or ralph-loop - Activates with plan. Choose based on task type:
    - ralph-loop: well-defined task, clear done condition, benefits from iteration. Use /cancel-ralph to stop.
    - subagent-driven-development: complex tasks needing fresh context per subtask, two-stage review.
    - executing-plans: sequential execution with human checkpoints.

    test-driven-development - Activates during implementation. Enforces RED-GREEN-REFACTOR: write failing test, watch it fail, write minimal code, watch it pass, commit. Deletes code written before tests.

    requesting-code-review - Activates between tasks. Reviews against plan, reports issues by severity. Critical issues block progress.

    verification-before-completion - Activates when all tasks done. Confirms tests pass, no regressions. Gate before merge.

    finishing-a-development-branch - Activates after verification. Presents options (merge/PR/keep/discard), cleans up worktree.

The agent checks for relevant skills before any task. Mandatory workflows, not suggestions.
