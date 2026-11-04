# Dev Cycle

Three OpenCode agents. One spec-driven pipeline: **Propose → Approve → Plan → Build → Verify → Archive**.

Only three files matter at runtime:

```
opencode.json
.opencode/agent/architect.md
.opencode/agent/executor.md
.opencode/agent/surgeon.md
```

`opencode.json` sets `architect` as the default agent.

## The agents

### @architect — orchestrator (`mode: primary`)

Runs as the main session agent. Does not write feature code. It:

- Routes each request: full pipeline for new features, direct to `@surgeon` for fixes and refactors.
- Creates and persists stage artifacts: `out/proposal.md`, `out/plan.md`, `out/tasks.md`, `out/verification.md`.
- Stops at confirmation gates: after PROPOSE, after PLAN, after BUILD.
- Dispatches `@executor` for multi-file features and `@surgeon` for isolated changes.
- Archives finished work from `out/` to `archive/<feature>/`.

Resumes partially-completed work by checking which artifacts already exist.

### @executor — feature builder (`mode: subagent`)

Implements features from a plan. Reads `out/plan.md` and `out/tasks.md`, then works through pending tasks in order.

- Writes production-ready code: no placeholders, no TODOs.
- Works test-first: failing test, implementation, refactor.
- Matches existing project style, linter, formatter.
- Marks each finished task `[x]` in `out/tasks.md`.
- Updates `out/PROJECT_MAP.md` when present.

### @surgeon — precise editor (`mode: subagent`)

Makes one isolated change: bug fix, behavior change, or focused refactor.

- Traces dependencies and callers before editing.
- Touches only what the change requires. No reformatting, no drive-by refactors.
- Writes a failing test, fixes, confirms no regression.
- Runs the project lint, typecheck, or build on changed files.
- Updates `out/tasks.md` or `out/PROJECT_MAP.md` when those exist.

## How to use

Full pipeline for a new feature:

```
@architect add login with Google OAuth
```

Direct fix, no gates:

```
@surgeon fix the null reference in PaymentService.process()
```

Implement a written spec:

```
@executor implement the tasks in out/tasks.md
```

## Models

Each agent declares its model in frontmatter. Current values:

| Agent | Model |
|---|---|
| architect | `opencode/big-pickle` |
| executor | `opencode/deepseek-v4.1-flash-free` |
| surgeon | `opencode/deepseek-v4.1-flash-free` |

Change the `model:` line in the agent file to switch. Use a model your provider actually lists.

## Artifacts

| Path | Written by | Contents |
|---|---|---|
| `out/proposal.md` | architect | requirements, scope, constraints |
| `out/plan.md` | architect | architecture, milestones |
| `out/tasks.md` | architect, updated by executor/surgeon | task checklist |
| `out/PROJECT_MAP.md` | executor/surgeon | system state, gaps |
| `out/verification.md` | architect | test results, spec compliance |
| `archive/<feature>/` | architect | closed-out artifacts |

`out/` and `archive/` are generated at runtime and gitignored.

## Adding an agent

Drop a Markdown file in `.opencode/agent/` (`agents/` also works):

```markdown
---
description: What this agent does
mode: subagent
model: opencode/deepseek-v4.1-flash-free
---
System prompt here.
```

The filename becomes the agent ID. `mode` is `primary`, `subagent`, or `all`.
