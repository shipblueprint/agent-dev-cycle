[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE) ![GitHub release](https://img.shields.io/github/v/release/shipblueprint/agent-dev-cycle)

# Dev Cycle — Spec-Driven Development Pipeline

**Multi-agent development cycle for OpenCode.** Enforces **Spec → Plan → Execute → Archive** discipline. No blind coding — every change starts with a spec and ends with an archive.

---

## Quick Start

```bash
cd /path/to/your/project
opencode
```

You land in `@architect` (the default orchestrator). From here you can:

**Use the full pipeline** (recommended for new features):

```
"add login with Google OAuth"
```

@architect walks you through: PROPOSE → APPROVE → PLAN → BUILD → VERIFY → ARCHIVE, dispatching the right subagent at each stage.

**Or call a specialist directly** (for precise work):

```
@surgeon fix the null reference in PaymentService.process()
@executor implement the user model with email/password auth
```

No pipeline, no gates — goes straight to work.

---

## Why only @architect shows up?

Each agent declares a `mode` in its config:

| mode | What it means |
|------|---------------|
| `primary` | Loaded automatically, visible as default agent |
| `subagent` | Not auto-loaded — dispatchable by `@name` in chat |

- **@architect** is `mode: primary` — loaded on enter.
- **@executor** and **@surgeon** are `mode: subagent` — not visible in the UI but fully callable.

In OpenCode, type `@executor` or `@surgeon` in chat to switch to them. OpenCode resolves `@name` mentions and routes the session to that agent.

---

## Two workflows

### Workflow A: Full pipeline (via @architect)

```
You describe what you want
        │
  @architect (default)
        │
  ├─ PROPOSE → out/proposal.md
  ├─ APPROVE ← you sign off
  ├─ PLAN    → out/plan.md + out/tasks.md
  ├─ BUILD   → dispatches @executor (features) or @surgeon (fixes)
  ├─ VERIFY  → out/verification.md
  └─ ARCHIVE → archive/<feature>/
```

@architect gates each stage — you approve before it proceeds.

### Workflow B: Direct call

```
You: "@surgeon fix the null reference in PaymentService"
                      │
              @surgeon skips straight to BUILD
              (impact analysis → edit → test → done)
```

No proposal, no plan, no gates. Use when you know exactly what needs to change.

---

## Which workflow to use

| When | Use |
|------|-----|
| New feature, multi-file change | Full pipeline → @architect |
| Bug fix, precise change | @surgeon directly |
| Refactor, optimization | @surgeon directly |
| Implementing from a written spec | @executor directly |
| Design/architecture discussion | @architect (PROPOSE + PLAN only) |
| Code review | @reviewer directly |

---

## All available agents

| Agent | Purpose | Invoke |
|-------|---------|--------|
| **@architect** | Orchestrator — routes requests, enforces the cycle | Default, or `@architect` |
| **@executor** | Full feature implementation with TDD | `@executor` |
| **@surgeon** | Surgical code modifications | `@surgeon` |
| **@reviewer** | Code review / quality gate (optional) | `@reviewer` |

---

## Pipeline stages

```
PROPOSE → APPROVE → PLAN → BUILD → VERIFY → ARCHIVE
```

| Stage | What happens | Artifact |
|-------|-------------|----------|
| **PROPOSE** | Understand intent, create proposal | `out/proposal.md` |
| **APPROVE** | Present to user, get sign-off | — |
| **PLAN** | Architecture design, task breakdown | `out/plan.md`, `out/tasks.md` |
| **BUILD** | @executor (features) or @surgeon (fixes) | Code changes |
| **VERIFY** | Run tests, spec compliance | `out/verification.md` |
| **ARCHIVE** | Close out, save artifacts | `archive/<feature>/` |

---

## Resume & partial runs

The pipeline detects already-completed stages by checking output files:

| Check | Means |
|-------|-------|
| `out/proposal.md` exists | Stage 1 done |
| `out/plan.md` exists | Stage 3 done |
| `out/tasks.md` with `[x]` | Stage 4 done |
| `out/verification.md` exists | Stage 5 done |
| `archive/<feature>/` exists | Fully archived |

---

## Architecture

```
┌──────────────────────────────────────────────┐
│               OpenCode Runtime                │
│  (agent dispatcher, model resolver, fs)       │
└──────────────────┬───────────────────────────┘
                   │
         ┌─────────▼──────────┐
         │   @architect        │
         │   (primary)         │
         │   Orchestrator      │
         └─────────┬──────────┘
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
    ┌────────┐┌────────┐┌────────┐
    │@executor││@surgeon││@reviewer│
    │subagent ││subagent││subagent │
    └────────┘└────────┘└────────┘
```

---

## Project structure

```
dev-cycle/
├── opencode.json              # default_agent: architect
├── AGENTS.md                  # Multi-IDE agent definitions
├── OPENCODE.md                # OpenCode runtime instructions
├── README.md                  # This file
├── .opencode/
│   ├── agent/
│   │   ├── architect.md       # Architecture: mode: primary
│   │   ├── executor.md        # Subagent: mode: subagent
│   │   └── surgeon.md         # Subagent: mode: subagent
│   ├── pipelines/
│   │   └── dev-cycle/
│   │       └── OPENCODE.md    # Pipeline stage definitions
│   └── skills/                # Reusable skills (add as needed)
└── out/                       # Stage artifacts (generated)
```

---

## Changing models

Each agent declares its model in YAML frontmatter. Edit the `model:` field in `.opencode/agent/*.md` to switch:

```yaml
# .opencode/agent/executor.md
---
name: executor
model: opencode/north-mini-code-free  # <-- change this
---
```

---

## Requirements

- [OpenCode](https://opencode.ai)
- Git (for project context)

---

## License

MIT
