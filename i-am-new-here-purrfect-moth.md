# Context

User is new to Claude Code and wants to understand its built-in orchestration primitives. They know PRDs, task files, and the ralph loop pattern (ghuntley.com/loop — single autonomous agent, iterative refinement, context engineering). Goal: map those concepts to Claude Code's actual primitives so they can start using them effectively.

---

# Ralph Loop → Claude Code Mapping

The ralph loop is: **allocate context → set goal → loop until done**. Claude Code implements this natively.

| Ralph Loop concept | Claude Code equivalent |
|---|---|
| "Allocate context" | Plan file + focused agent prompts (context engineering) |
| "Set goal" | `/plan` → writes a plan file with clear scope |
| "Loop until done" | `superpowers:executing-plans` — runs with checkpoints |
| "Observe failure, fix, prevent recurrence" | `superpowers:systematic-debugging` skill |
| "Verify before claiming success" | `superpowers:verification-before-completion` skill |
| PRD | Input to `superpowers:writing-plans` → produces a plan file |
| Task file | Plan file + TaskCreate entries generated during execution |
| Iterative refinement | `/loop [interval] /prompt` — recurring autonomous runs |

---

# The Five Primitives

### 1. Skills (`/skill-name` or `Skill` tool)
Named processes that override how Claude approaches a task. The **process layer** — tells Claude HOW to work.

Key skills for ralph-loop-style work:

| Skill | When to use |
|---|---|
| `superpowers:brainstorming` | Before building anything — explore intent first |
| `superpowers:writing-plans` | Turn a PRD/spec into a plan file |
| `superpowers:executing-plans` | Execute a plan with review checkpoints |
| `superpowers:systematic-debugging` | Any bug or unexpected behavior |
| `superpowers:verification-before-completion` | Before claiming anything is done |
| `superpowers:test-driven-development` | Tests before implementation |
| `/loop [interval]` | Recurring autonomous work (pure ralph loop mode) |

### 2. Subagents (`Agent` tool)
Claude spawns specialized child agents with zero inherited context — you give each agent exactly what it needs (this IS context engineering). Types:

- `Explore` — read-only codebase search
- `Plan` — design an approach
- `general-purpose` — research, implement, multi-step tasks

**Parallel dispatch**: multiple `Agent` calls in one message run concurrently. Use for independent sub-problems.

### 3. Plan files
Markdown written during `/plan` mode, stored in `~/.claude/plans/`. The persistent artifact that bridges planning → execution sessions.

### 4. Tasks (`TaskCreate` / `TaskUpdate` / `TaskList`)
In-session checklist Claude maintains as it executes. Auto-created from plan checklists.

### 5. `/loop`
Pure ralph loop: `/loop 5m /check-deploy` runs that command every 5 minutes. Omit the interval for self-paced autonomous looping.

---

# Typical Workflow (PRD → Done)

```
You have a PRD or spec
    ↓
/plan
  Claude enters plan mode:
  - Spawns Explore agents to read the codebase
  - Spawns Plan agent to design the approach
  - Asks clarifying questions
  - Writes plan file → your approval
    ↓
You approve
  superpowers:executing-plans takes over:
  - Reads plan file
  - Spawns subagents per task (parallel where independent)
  - Tracks progress with Tasks
  - Review checkpoints between phases
    ↓
superpowers:verification-before-completion
  - Runs tests, checks output
  - No success claims without evidence
    ↓
superpowers:requesting-code-review → PR or merge
```

---

# Context Engineering in Claude Code

The ralph loop's key discipline is context engineering — giving the agent exactly what it needs and nothing else. In Claude Code:

- **Plan mode** forces exploration before acting so the plan file captures relevant context
- **Subagent prompts** are hand-crafted with specific scope, constraints, and expected output
- **Skills** encode proven context recipes so you don't have to re-engineer each time

---

# What to Try First

1. **Start any non-trivial task with `/plan`** — forces the explore → design → approve loop
2. **Watch skill announcements** — Claude says "Using [skill] to [purpose]" before each
3. **Use `/loop` for autonomous iteration** — give it a goal, let it run
4. **Give Claude a small PRD** and watch the full cycle execute

---

# Verification

Conceptual orientation — no code changes. Success = user can map their existing mental model (ralph loop, PRD, task files) to Claude Code primitives and knows which command to reach for first.
