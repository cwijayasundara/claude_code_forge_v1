# claude-code-forge

A Claude Code plugin that turns natural language into production-ready code through an 11-phase SDLC pipeline. Agents write all code. Specs are the source of truth. Humans approve at two checkpoints.

**20 specialized agents | 23 slash commands | 23 skills | 11 hook scripts**

---

## Table of Contents

- [Plugin Architecture](#plugin-architecture)
- [What It Does](#what-it-does)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [How It Works](#how-it-works)
  - [Request Routing](#request-routing)
  - [The SDLC Pipeline](#the-sdlc-pipeline)
  - [TDD Enforcement](#tdd-enforcement)
  - [Validation System](#validation-system)
  - [Team Orchestration](#team-orchestration)
- [Architecture](#architecture)
  - [6-Layer Model](#6-layer-model)
  - [Agents](#agents)
  - [Skills](#skills)
  - [Hook System](#hook-system)
- [Commands Reference](#commands-reference)
- [Workflows](#workflows)
  - [Build a New App from Scratch](#build-a-new-app-from-scratch)
  - [Add a Feature to an Existing App](#add-a-feature-to-an-existing-app)
  - [Quick Build Without Spec Ceremony](#quick-build-without-spec-ceremony)
  - [Resume an Interrupted Pipeline](#resume-an-interrupted-pipeline)
  - [Debug a Problem](#debug-a-problem)
  - [Review Code Quality](#review-code-quality)
  - [Reverse-Engineer Docs from Code](#reverse-engineer-docs-from-code)
- [Project Structure](#project-structure)
- [Configuration](#configuration)
- [Extending Forge](#extending-forge)
- [Troubleshooting](#troubleshooting)
- [Development](#development)

---

## Plugin Architecture

How all plugin components connect to Claude Code at a glance:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                            CLAUDE CODE (CLI)                           │
│                                                                        │
│   Loads at startup     ┌──────────────────────────────┐                │
│   ──────────────────►  │         CLAUDE.md            │                │
│                        │   (Project Constitution)     │                │
│                        │                              │                │
│                        │  • Request routing rules     │                │
│                        │  • 6-layer architecture      │                │
│                        │  • TDD enforcement           │                │
│                        │  • Quality gates              │                │
│                        └──────────────┬───────────────┘                │
│                                       │                                │
│                              Routes to .claude/                        │
│                                       │                                │
│          ┌────────────────────────────┼────────────────────────────┐   │
│          │                            │                            │   │
│          ▼                            ▼                            ▼   │
│  ┌───────────────┐          ┌─────────────────┐          ┌──────────┐ │
│  │   Commands    │          │     Skills      │          │  Agents  │ │
│  │   (23)        │          │     (24)        │          │  (20)    │ │
│  │               │          │                 │          │          │ │
│  │ User-facing   │ invoke   │  Composable     │ loaded   │Specialized│ │
│  │ /forge:* ─────┼────────► │  workflow steps │◄────────┤ actors   │ │
│  │ entry points  │          │                 │  by      │ per phase│ │
│  └───────────────┘          └─────────────────┘          └──────────┘ │
│          │                          │                         │        │
│          │                          │                         │        │
│          ▼                          ▼                         ▼        │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │                    Runtime Enforcement                          │  │
│  │                                                                 │  │
│  │  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │  │
│  │  │  Hooks   │    │ Linters  │    │ Scripts  │    │  Evals   │  │  │
│  │  │  (11)    │    │  (3)     │    │  (3)     │    │  (6+)    │  │  │
│  │  │          │    │          │    │          │    │          │  │  │
│  │  │Pre/Post  │    │Layer deps│    │Scaffold  │    │Good/bad  │  │  │
│  │  │tool-use  │    │File size │    │Validate  │    │code for  │  │  │
│  │  │guards    │    │checks    │    │Migrate   │    │reviewer  │  │  │
│  │  └──────────┘    └──────────┘    └──────────┘    │calibration│ │  │
│  │                                                   └──────────┘  │  │
│  └─────────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │                    Supporting Infrastructure                    │  │
│  │                                                                 │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │  │
│  │  │Settings  │  │   MCP    │  │Templates │  │    Docs      │   │  │
│  │  │(.json)   │  │ Servers  │  │(.md+code)│  │(architecture │   │  │
│  │  │          │  │          │  │          │  │ conventions  │   │  │
│  │  │Permissions│ │Playwright│  │Spec &    │  │ testing      │   │  │
│  │  │Env vars  │  │Stitch    │  │project   │  │ pipeline)    │   │  │
│  │  └──────────┘  └──────────┘  │scaffolds │  └──────────────┘   │  │
│  │                               └──────────┘                     │  │
│  └─────────────────────────────────────────────────────────────────┘  │
│                                                                        │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## What It Does

You describe what you want in plain English. Forge handles the rest:

1. **Interviews you** to clarify requirements (Socratic dialogue, not a form)
2. **Writes a spec** with acceptance criteria, data models, API contracts
3. **Decomposes into stories** with a dependency graph and parallel groups
4. **Creates a design doc** with layer impact analysis and sequence diagrams
5. **Generates a test plan** with concrete test data and coverage targets
6. **Produces an execution plan** with file paths and implementation order
7. **Waits for your approval** before writing any code
8. **Implements with TDD** — every acceptance criterion gets a failing test first, then minimum code to pass, then refactor
9. **Validates with 8 parallel reviewers** — spec compliance, code quality, security, performance, architecture alignment
10. **Waits for your approval again** before creating a PR
11. **Creates a structured PR** with story-based commits

When there are 4+ stories with 2+ parallel groups, forge automatically spins up **agent teams** — multiple independent Claude Code sessions that implement stories in parallel, coordinating through a shared task list.

---

## Installation

### Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- Python 3.10+ (for linters and hook scripts)
- Git (for branch management and PRs)
- `pytest` (for test execution): `pip install pytest pytest-cov`

### How it works

You use `--plugin-dir` **once** to bootstrap a new project. The `/scaffold` command copies the entire forge toolkit (agents, commands, skills, hooks, linters, scripts) into your project's `.claude/` directory. After that, you just run `claude` — no `--plugin-dir` needed.

```
# Before /scaffold:                 # After /scaffold:
my-project/                        my-project/
└── .git/                          ├── .claude/          ← full forge toolkit
                                   │   ├── agents/       (20 agents)
                                   │   ├── commands/     (23 commands)
                                   │   ├── skills/       (23 skills)
                                   │   ├── hooks/        (11 hooks)
                                   │   ├── linters/
                                   │   ├── scripts/
                                   │   └── ...
                                   ├── src/              ← 6-layer architecture
                                   ├── tests/
                                   ├── specs/
                                   ├── CLAUDE.md
                                   └── README.md
```

### Step-by-step setup for a new project

**Step 1 — Clone the forge (one-time, shared across all projects):**

```bash
git clone https://github.com/cwijayasundara/claude_code_forge_v1 ~/claude-code-forge
```

**Step 2 — Create and initialize your new project:**

```bash
mkdir my-project && cd my-project
git init
```

**Step 3 — Bootstrap with `--plugin-dir` (one-time only for this project):**

```bash
claude --plugin-dir ~/claude-code-forge/.claude
```

This loads the forge as a temporary plugin. Commands appear with a namespace prefix (e.g., `/claude-code-forge:scaffold`). You only need this once — to run `/scaffold`.

> **Note:** The plugin directory is `.claude/` inside the forge repo — not the repo root. Always point `--plugin-dir` to `~/claude-code-forge/.claude`.

**Step 4 — Inside the Claude session, scaffold everything:**

```
> /claude-code-forge:scaffold python-fastapi
```

This copies the full forge toolkit into your project's `.claude/` directory, creates the 6-layer `src/` structure, `tests/`, `specs/`, `CLAUDE.md`, `README.md`, and all config files. Hook script paths are automatically rewritten from plugin paths to local relative paths.

**Step 5 — Exit and restart Claude without `--plugin-dir`:**

```
> /exit
claude
```

Since the entire `.claude/` directory now lives inside your project, Claude Code auto-discovers all commands, agents, skills, and hooks locally. No `--plugin-dir` flag needed — ever again for this project.

Commands now appear without the namespace prefix: `/build` instead of `/claude-code-forge:build`.

**Step 6 — Start building:**

```
> /build a task management API with user authentication
```

> **Note:** `/scaffold` creates the project **structure** only — directories, config files, and the forge toolkit. It does NOT generate specs or write application code. To execute the full 11-phase SDLC pipeline (specs, stories, design, test plan, TDD implementation, review), run `/build`. If you want a quick build without spec ceremony, use `/just-do-it`.

**For team members** who clone your project later, they just run `claude` — the `.claude/` directory is already in the repo with everything they need.

### What `/scaffold` copies

| Destination | Contents |
|-------------|----------|
| `.claude/agents/` | 20 specialized agents (spec-writer, implementer, reviewer, etc.) |
| `.claude/commands/` | 23 slash commands (/build, /review, /debug, etc.) |
| `.claude/skills/` | 23 skills (TDD, validation, architecture, etc.) |
| `.claude/hooks/` | 11 hook scripts (auto-lint, commit alignment, etc.) |
| `.claude/linters/` | Layer dependency + file size linters |
| `.claude/scripts/` | Scaffold and utility scripts |
| `.claude/templates/` | Spec and story templates |
| `.claude/settings.json` | Default permissions and env vars |
| `.claude/.mcp.json` | MCP servers (Playwright for E2E, Stitch for Google Cloud) |
| `CLAUDE.md` | Project instructions with architecture + routing rules |
| `README.md` | Project readme with setup and command reference |

### Validating the plugin source

Before first use, you can verify the forge plugin structure:

```bash
claude plugin validate ~/claude-code-forge/.claude
# Expected: ✔ Validation passed
```

### Available commands

After installation, all forge commands become available in your Claude Code session.

> **Note:** When using forge commands, you type them **without** the `forge:` prefix. For example, use `/scaffold` instead of `/forge:scaffold`. Here are all the available commands:
>
> | Command | Description |
> |---------|-------------|
> | `/build` | Full 11-phase SDLC pipeline |
> | `/add-feature` | Feature SDLC pipeline for existing apps |
> | `/resume` | Resume pipeline from last checkpoint |
> | `/design` | Spec interview only (no implementation) |
> | `/design-product` | Product spec design workflow |
> | `/design-architecture` | Architecture design workflow |
> | `/just-do-it` | Quick build without spec ceremony |
> | `/build-unplanned` | Lightweight build with brief |
> | `/work-on` | Work on a specific task/story (TDD) |
> | `/work-on-next` | Pick next unblocked story |
> | `/create-tasks` | Create tasks from stories |
> | `/review` | 8-agent parallel gating review |
> | `/validate` | Full verification suite |
> | `/check` | 3-dimension alignment checks |
> | `/source-architecture` | Architecture docs from code |
> | `/source-specs` | Specs from existing code |
> | `/spec-sync` | Bidirectional spec-code sync |
> | `/scaffold` | Scaffold project structure (copies full forge toolkit) |
> | `/debug` | 5-phase debugging workflow |
> | `/refactor` | Targeted debt reduction |
> | `/create-pr` | Structured PR with story commits |
> | `/skills` | List all available skills |
> | `/help` | Show all commands, skills, agents |

---

## Quick Start

### Build a new app

```
> /build a task management API with user authentication and team workspaces
```

Forge interviews you, produces a spec, waits for approval, then implements everything with TDD and full review.

### Add a feature

```
> /add-feature add real-time notifications via WebSocket
```

Same pipeline, scoped to a single feature.

### Quick change (skip the spec)

```
> /just-do-it add a health check endpoint at GET /health
```

Builds with TDD but skips the full spec ceremony.

### Resume after interruption

```
> /resume
```

Reads `specs/pipeline_status.md` and picks up where it left off.

---

## How It Works

### Request Routing

Every message you send is automatically classified by the routing skill:

```
┌─────────────────────────────────────────────────────────────────┐
│                        Your Message                             │
└────────────┬────────────────┬────────────────┬─────────────────┘
             │                │                │
     "Build me X"      "Just do it"     "Continue"      Everything else
     "Add feature X"   "Quick change"   "Resume"        (bugs, questions,
     "Create a Y app"  "Skip the spec"  "What's next"    refactoring)
             │                │                │                │
             ▼                ▼                ▼                ▼
      Full Pipeline      Quick Build       Resume          Toolbox
      (11 phases)     (TDD, no spec)    (from last       (23 commands
                                         checkpoint)      available)
```

| Trigger Words | Mode | What Happens |
|--------------|------|--------------|
| "build me", "create a", "add feature", "new app" | **Pipeline** | Full 11-phase SDLC with spec interview |
| "just do it", "quick change", "skip the spec" | **Quick Build** | TDD implementation without spec ceremony |
| "continue", "resume", "what's next" | **Resume** | Reads pipeline status, continues from last phase |
| Everything else | **Toolbox** | Individual commands for debugging, reviewing, refactoring |

### The SDLC Pipeline

The pipeline has 11 phases with two human approval checkpoints:

```
Phase 1    Phase 2     Phase 3    Phase 4      Phase 5
 SPEC   →  STORIES  →  DESIGN  → TEST PLAN → EXEC PLAN
  │          │           │          │            │
  └──────────┴───────────┴──────────┴────────────┘
                    spec-writer agent
                    (Socratic interview)
                           │
                    ┌──────┴──────┐
                    │  APPROVE?   │  ◄── You review the plan
                    └──────┬──────┘
                           │
Phase 6       Phase 7     Phase 8    Phase 9
IMPLEMENT  →  TEST FILL → E2E     → DEVOPS
  │              │          │          │
  │ (TDD)       │          │          │
  │ (teams?)    │          │          │
  └──────────────┴──────────┴──────────┘
                           │
Phase 10                   │
 REVIEW  ◄────────────────┘
  │
  │ (8 parallel reviewers)
  │
  ┌──────┴──────┐
  │  APPROVE?   │  ◄── You review the results
  └──────┬──────┘
         │
Phase 11 │
   PR  ◄─┘
```

#### Phase-by-Phase Breakdown

| Phase | Agent | What Happens | Output |
|-------|-------|--------------|--------|
| 1. Spec | spec-writer | Socratic interview to clarify requirements | `specs/features/<name>.md` |
| 2. Stories | spec-writer | Decompose into user stories with dependencies | `specs/stories/<name>.md` |
| 3. Design | spec-writer | Layer impact analysis, API contracts, sequence diagrams | `specs/design/<name>.md` |
| 4. Test Plan | spec-writer | Test cases with concrete data, coverage targets | `specs/tests/<name>.md` |
| 5. Exec Plan | spec-writer | Implementation tasks, file paths, ordering | `specs/plans/<name>.md` |
| **Checkpoint 1** | *you* | **Review and approve the plan** | |
| 6. Implement | implementer / team | TDD per acceptance criterion, story-by-story | `src/` + `tests/` |
| 7. Test Fill | test-writer | Fill coverage gaps to reach 80%+ | `tests/` |
| 8. E2E | e2e-writer | Browser tests (Playwright) + API contract tests | `tests/e2e/` |
| 9. DevOps | devops | CI/CD, Dockerfile, infrastructure | `infra/`, `.github/workflows/` |
| 10. Review | 8 reviewers | Parallel validation with unanimous approval | `specs/review_report.md` |
| **Checkpoint 2** | *you* | **Review results and approve for PR** | |
| 11. PR | pr-writer | Structured pull request with story-based commits | GitHub PR |

Each phase records its status in `specs/pipeline_status.md`. If a session is interrupted, `/resume` reads this file and continues from the next incomplete phase.

#### Spec Artifacts

After phases 1-5, your `specs/` directory contains:

```
specs/
├── app_spec.md                  # (greenfield only) Full app specification
├── pipeline_status.md           # Pipeline progress tracker
├── features/
│   └── <feature-name>.md        # Feature spec with acceptance criteria
├── stories/
│   └── <feature-name>.md        # User stories with dependency graph
├── design/
│   └── <feature-name>.md        # Design doc with API contracts
├── tests/
│   └── <feature-name>.md        # Test plan with test cases (TC-XXX)
└── plans/
    └── <feature-name>.md        # Execution plan with tasks and file paths
```

### TDD Enforcement

Every acceptance criterion must go through the TDD cycle. This is enforced by the `test-driven-development` skill and is not optional.

```
For each acceptance criterion:

  1. RED    ─── Write a failing test that asserts the expected behavior
             └─ Run: pytest tests/<file>::<test> -x -q
             └─ Verify: fails with assertion error (not import/syntax error)

  2. GREEN  ─── Write MINIMUM code to make the test pass
             └─ No code for other criteria or future needs
             └─ Run: pytest tests/<file>::<test> -x -q
             └─ Verify: test passes

  3. REFACTOR ─ Clean up duplication, improve naming
               └─ Run: pytest tests/ -x -q (full suite)
               └─ Verify: all tests still pass
```

After each story is implemented, a 3-checker validation loop runs automatically:
1. **architecture-alignment-checker** — does the code match the architecture docs?
2. **design-consistency-checker** — does the UI follow the design system?
3. **prd-architecture-checker** — do the architecture docs match the requirements?

### Validation System

Forge uses a three-tier verification system:

#### Tier 1: Gating Reviews (Phase 10 — blocks PR)

8 reviewer agents run **in parallel**. The 5 gating reviewers must all approve unanimously:

| # | Reviewer | Type | What It Checks |
|---|----------|------|----------------|
| 1 | spec-reviewer | **Gating** | Every acceptance criterion has implementation + tests |
| 2 | code-reviewer | **Gating** | Naming conventions, file size, type hints, logging, imports |
| 3 | security-reviewer | **Gating** | OWASP Top 10, auth flows, hardcoded secrets, input validation |
| 4 | performance-reviewer | **Gating** | N+1 queries, unbounded loops, missing pagination, blocking I/O |
| 5 | architecture-alignment-checker | **Gating** | Layer placement, dependency direction, service isolation |
| 6 | design-consistency-checker | Advisory | UI compliance with design system |
| 7 | code-simplifier | Advisory | Over-engineering, unnecessary abstractions |
| 8 | prd-architecture-checker | Advisory | Architecture docs match requirements |

If any gating reviewer returns `REQUEST_CHANGES`:
1. Findings are collected from all failing reviewers
2. The implementer fixes the issues
3. Only the failing reviewers re-run
4. Max 3 retry cycles before escalating to you

#### Tier 2: Advisory Checks

Non-blocking checks included in the review report:
- Test coverage report (target: 80%+)
- Linter results (layer deps + file size)
- Dependency audit (CVEs)
- Documentation coverage
- Deployment readiness (Dockerfile builds, CI syntax)

#### Tier 3: Per-Story Validation

Runs after each story during implementation (not just at the end):
- architecture-alignment-checker
- design-consistency-checker (if UI changes)
- prd-architecture-checker

### Team Orchestration

When forge detects **4+ stories AND 2+ parallel groups**, it automatically activates Claude Code agent teams. This is a mechanical check — it cannot be overridden based on subjective judgment.

> Agent teams are experimental and require `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` (pre-configured in `.claude/settings.json`).

```
                    ┌─────────────┐
                    │   Lead      │  (main conversation)
                    │  Session    │
                    └──────┬──────┘
                           │
              TeamCreate + TaskCreate
              (one task per story)
                           │
           ┌───────────────┼───────────────┐
           │               │               │
    ┌──────┴─────┐  ┌──────┴─────┐  ┌──────┴─────┐
    │ Teammate 1 │  │ Teammate 2 │  │ Teammate 3 │
    │ (Group A)  │  │ (Group B)  │  │ (Group C)  │
    └──────┬─────┘  └──────┬─────┘  └──────┬─────┘
           │               │               │
      Self-claim      Self-claim      Self-claim
      tasks from      tasks from      tasks from
      shared list     shared list     shared list
           │               │               │
       TDD cycle       TDD cycle       TDD cycle
       per story       per story       per story
           │               │               │
      SendMessage     SendMessage     SendMessage
      (completion)    (completion)    (completion)
           │               │               │
           └───────────────┼───────────────┘
                           │
                    Lead verifies
                    Full test suite
                    TeamDelete
```

**Key concepts:**

- Each teammate is a **full, independent Claude Code session** with its own context window
- Teammates automatically load CLAUDE.md, MCP servers, and skills — but **NOT** the lead's conversation history
- Teammates self-claim tasks using file locking (no race conditions)
- Task dependencies auto-unblock when predecessors complete
- Teammates can message each other directly (not just the lead)
- **File ownership** prevents conflicts: each task description lists which files that teammate may modify
- Teammates inherit the lead's permission settings at spawn time

**Quality gate hooks enforce standards automatically:**

| Hook | When It Fires | What It Does |
|------|---------------|--------------|
| `TeammateIdle` | Teammate about to go idle | Checks for incomplete owned tasks. Exit code 2 keeps the teammate working. |
| `TaskCompleted` | Task being marked complete | Runs linters and tests. Exit code 2 rejects the completion. |

**Team lifecycle:**

```
1. TeamCreate      → creates team + shared task list
2. TaskCreate      → one task per story with dependency blocking
3. Task (spawn)    → one implementer agent per parallel group
4. Monitor         → TaskList polling + SendMessage notifications
5. Verify          → full test suite + linters after all tasks complete
6. Shutdown        → shutdown_request to each teammate → wait for responses
7. TeamDelete      → cleanup (only the lead runs this)
```

**Display modes:**

Agent teams support two display modes, configured via the `teammateMode` user setting or `--teammate-mode` CLI flag:

| Mode | Description | Requirements |
|------|-------------|-------------|
| `in-process` | All teammates run in the main terminal. `Shift+Down` to cycle between them. | Any terminal |
| `tmux` | Each teammate gets its own split pane. Click to interact. | tmux or iTerm2 with `it2` CLI |
| `auto` (default) | Uses tmux if already in a tmux session, otherwise in-process | Varies |

To set the display mode for a session:

```bash
claude --teammate-mode tmux
```

Or in your **user-level** `~/.claude/settings.json` (not the project settings):

```json
{
  "teammateMode": "in-process"
}
```

Split-pane mode requires [tmux](https://github.com/tmux/tmux/wiki) or iTerm2 with the [`it2` CLI](https://github.com/mkusaka/it2). It is **not supported** in VS Code's integrated terminal, Windows Terminal, or Ghostty.

**Interacting with teammates directly:**

- **In-process mode**: `Shift+Down` to cycle through teammates, type to message them. `Enter` to view a session, `Escape` to interrupt their turn. `Ctrl+T` to toggle the task list.
- **Split-pane mode**: Click into a teammate's pane to interact with their session directly.

**Plan approval for teammates:**

For complex stories, you can require teammates to plan before implementing by spawning them with `mode: "plan"`. The teammate works in read-only plan mode until the lead approves their approach.

**Token costs:**

Agent teams use significantly more tokens than a single session. Each teammate has its own context window and token usage scales linearly with the number of active teammates. Start with 3-5 teammates. For routine tasks, a single session is more cost-effective.

**Known limitations:**

| Limitation | Details |
|-----------|---------|
| No session resumption for teammates | `/resume` and `/rewind` do not restore in-process teammates. After resuming, spawn new teammates. |
| Task status can lag | Teammates sometimes fail to mark tasks complete, blocking dependents. Check manually and update via TaskUpdate. |
| Shutdown can be slow | Teammates finish their current request before shutting down. |
| One team per session | Clean up the current team before starting a new one. |
| No nested teams | Teammates cannot spawn their own teams. |
| Lead is fixed | The session that creates the team is the lead for its lifetime. You cannot promote a teammate to lead. |
| Permissions set at spawn | All teammates start with the lead's permission mode. You can change individual modes after spawning but not at spawn time. |

When teams are **not** required (fewer than 4 stories or only 1 parallel group), forge uses a single implementer agent that processes stories sequentially.

---

## Architecture

### 6-Layer Model

All projects built by forge follow a strict 6-layer architecture with forward-only dependencies:

```
┌─────────────────────────────────────────────┐
│  Layer 5: UI                                │
│  src/ui/   (routes, CLI, response formats)  │
├─────────────────────────────────────────────┤
│  Layer 4: Runtime                           │
│  src/runtime/  (server bootstrap, DI, MW)   │
├─────────────────────────────────────────────┤
│  Layer 3: Service                           │
│  src/service/  (business logic, no I/O)     │
├─────────────────────────────────────────────┤
│  Layer 2: Repo                              │
│  src/repo/  (DB, HTTP, filesystem access)   │
├─────────────────────────────────────────────┤
│  Layer 1: Config                            │
│  src/config/  (settings, env parsing)       │
├─────────────────────────────────────────────┤
│  Layer 0: Types                             │
│  src/types/  (Pydantic models, enums)       │
└─────────────────────────────────────────────┘
        ▲ Dependencies flow upward only ▲
```

| Layer | May Import From | Must NOT Import From |
|-------|-----------------|----------------------|
| Types (0) | nothing (foundation) | Config, Repo, Service, Runtime, UI |
| Config (1) | Types | Repo, Service, Runtime, UI |
| Repo (2) | Types, Config | Service, Runtime, UI |
| Service (3) | Types, Config, Repo | Runtime, UI |
| Runtime (4) | Types, Config, Repo, Service | UI |
| UI (5) | all layers | (top of stack) |

**Intentional constraints:**

- **Service has no direct I/O** — all database, HTTP, and filesystem access goes through Repo
- **Types has no project imports** — standard library and Pydantic only
- **No raw primitives for domain concepts** — use refined Pydantic types (`UserId`, `Email`, not bare `str`)
- **No global mutable state** — config is loaded once and injected forward

Layer rules are enforced by `.claude/linters/layer_deps.py` on every file write.

### Agents

20 specialized agents, each with a single responsibility:

#### Planning Agents

| Agent | What It Does |
|-------|-------------|
| `spec-writer` | Conducts Socratic interviews, produces specs, stories, design docs, test plans, and execution plans |
| `pipeline-orchestrator` | Manages phase transitions and gates across the 11-phase pipeline |

#### Implementation Agents

| Agent | What It Does |
|-------|-------------|
| `implementer` | Implements stories with TDD, works solo or as a team lead/member |
| `task-executor` | Executes individual tasks with strict TDD and post-task validation |
| `test-writer` | Fills coverage gaps after implementation (target: 80%+) |
| `e2e-writer` | Creates Playwright browser tests and httpx API contract tests |
| `devops` | Generates CI/CD pipelines, Dockerfiles, and infrastructure configs |

#### Review Agents (Gating — block PR)

| Agent | What It Checks |
|-------|---------------|
| `spec-reviewer` | Implementation matches every acceptance criterion in the spec |
| `code-reviewer` | Naming, file size, type hints, logging, imports, dead code |
| `security-reviewer` | OWASP Top 10, auth flows, secrets, input validation |
| `performance-reviewer` | N+1 queries, unbounded loops, pagination, caching, blocking I/O |
| `architecture-alignment-checker` | Code matches architecture docs, layer rules respected |

#### Review Agents (Advisory — inform only)

| Agent | What It Checks |
|-------|---------------|
| `design-consistency-checker` | UI follows the design system |
| `code-simplifier` | Over-engineering, unnecessary abstractions |
| `prd-architecture-checker` | Architecture docs align with product requirements |

#### Utility Agents

| Agent | What It Does |
|-------|-------------|
| `pr-writer` | Creates structured PRs with story-based commits and review summaries |
| `refactorer` | Targeted technical debt reduction (structure only, no behavior changes) |
| `research-agent` | Read-only codebase analysis and exploration |
| `debug-agent` | Systematic 5-phase debugging (reproduce, isolate, hypothesize, fix, verify) |
| `housekeeper` | Removes dead code, unused imports, stale TODOs |

### Skills

Skills are reusable capabilities that agents and commands invoke. Auto skills activate implicitly; manual skills are invoked explicitly.

| Skill | Type | Purpose |
|-------|------|---------|
| `routing` | Auto | Classifies requests into pipeline / quick-build / resume / toolbox |
| `layer-enforcement` | Auto | Enforces 6-layer architecture rules on every file write |
| `conventions` | Auto | Enforces naming, logging, file size, type hint rules |
| `test-driven-development` | Auto | Enforces RED-GREEN-REFACTOR cycle for every acceptance criterion |
| `understanding-feature-requests` | Auto | Parses natural language into structured feature briefs |
| `sdlc-pipeline` | Manual | Orchestrates the full 11-phase pipeline |
| `teams` | Manual | Full team lifecycle: TeamCreate, TaskCreate, spawn, monitor, shutdown |
| `validation-loop` | Manual | 8 parallel reviewers with unanimous gating approval |
| `task-validation-loop` | Manual | Post-task 3-checker validation |
| `implement-feature` | Manual | Feature implementation orchestrator (checks team threshold) |
| `execute-task` | Manual | Single task execution with TDD + validation |
| `next-task` | Manual | Picks the next unblocked, unassigned task |
| `tasks` | Manual | Task management — create from stories, list, update, track |
| `build-unplanned-feature` | Manual | Quick feature with TDD but without full spec ceremony |
| `verification-suite` | Manual | 3-tier verification: gating, advisory, per-story |
| `spec-sync-engine` | Manual | Bidirectional spec-code synchronization |
| `worktree-isolation` | Manual | Git worktree isolation for parallel features |
| `architecture` | Manual | Architecture docs management and validation |
| `design-system` | Manual | Design system management and enforcement |
| `check-alignment` | Manual | 3-dimension alignment: code/spec, code/arch, arch/requirements |
| `sync-architecture` | Manual | Reverse-engineer architecture docs from existing code |
| `sync-design-system` | Manual | Discover UI patterns from code, generate design system docs |
| `debugging` | Manual | Structured 5-phase debugging workflow |

### Hook System

Hooks run automatically at specific lifecycle events. They enforce quality gates without manual intervention.

| Event | Matcher | Script | What It Does |
|-------|---------|--------|--------------|
| `PreToolUse` | Write/Edit | `pre_write_check.py` | Validates file path and layer rules before writing |
| `PreToolUse` | Read | `pre_read_scaffold_guard.py` | Guards against reading scaffold internals |
| `PostToolUse` | Write/Edit | `post_write_lint.py` | Runs linters after every file write |
| `PostToolUse` | Bash | `commit_alignment.py` | Checks commit messages reference spec/story IDs |
| `PostToolUse` | Task | `subagent_validate.py` | Validates subagent outputs |
| `PostToolUse` | Team/Task tools | `team_activity_log.py` | Logs team events to `specs/team_activity.log` |
| `TeammateIdle` | * | `teammate_idle_check.py` | Blocks idle if teammate has incomplete tasks |
| `TaskCompleted` | * | `task_completed_check.py` | Blocks completion if linters/tests fail |
| `Stop` | (all) | `post_commit_spec_check.py` | Checks spec consistency at session end |
| (startup) | - | `session_start.py` | Initializes session context |
| (compact) | - | `pre_compact_save.py` | Saves context before conversation compaction |

---

## Commands Reference

Commands are invoked **without** the `forge:` prefix — just use `/build`, `/debug`, `/review`, etc.

### Pipeline & Build

| Command | Description | When to Use |
|---------|-------------|-------------|
| `/build <description>` | Full 11-phase SDLC pipeline | Building a new app from scratch |
| `/add-feature <description>` | Feature pipeline for existing apps | Adding a significant feature |
| `/just-do-it <description>` | TDD implementation, no spec ceremony | Small, well-understood changes |
| `/build-unplanned <description>` | Lightweight build with feature brief | Medium changes that need some structure |
| `/resume` | Resume from last checkpoint | After session interruption |

### Design & Architecture

| Command | Description | When to Use |
|---------|-------------|-------------|
| `/design <description>` | Spec interview only (no implementation) | When you want to plan without building |
| `/design-product <description>` | Product spec design workflow | Defining product requirements |
| `/design-architecture` | Architecture design workflow | Designing system architecture |

### Implementation

| Command | Description | When to Use |
|---------|-------------|-------------|
| `/work-on <story-id>` | Work on a specific task/story with TDD | Implementing a particular story |
| `/work-on-next` | Pick next unblocked story, implement | Continuing implementation sequentially |
| `/create-tasks` | Create tasks from stories via TaskCreate | Setting up team task list |

### Quality & Validation

| Command | Description | When to Use |
|---------|-------------|-------------|
| `/review` | Run the 8-agent parallel review | Checking code quality before PR |
| `/validate` | Full verification suite (all 3 tiers) | Comprehensive quality check |
| `/check` | 3-dimension alignment checks | Quick alignment verification |

### Reverse Engineering

| Command | Description | When to Use |
|---------|-------------|-------------|
| `/source-architecture` | Generate architecture docs from code | Documenting an existing codebase |
| `/source-specs` | Generate specs from existing code | Creating specs for undocumented code |
| `/spec-sync` | Bidirectional spec-code sync | Keeping specs and code in sync |

### Utilities

| Command | Description | When to Use |
|---------|-------------|-------------|
| `/scaffold <template>` | Scaffold project structure (copies full forge toolkit) | Starting a new project |
| `/debug <description>` | Systematic 5-phase debugging | Investigating a bug |
| `/refactor <description>` | Targeted debt reduction | Improving code structure |
| `/create-pr` | Create structured PR with story commits | After all reviews pass |
| `/skills` | List all available skills | Discovering capabilities |
| `/help` | Show all commands, skills, agents | Getting oriented |

---

## Workflows

### Build a New App from Scratch

```
You:    /build a recipe sharing platform with user profiles and social features

Forge:  [Phase 1 — SPEC]
        Conducts a 3-round Socratic interview:
        Round 1: Vision — What does it do? Who uses it?
        Round 2: Technical — Frontend/backend? Tech choices?
        Round 3: Scope — MVP features? What's v2?

        Produces: specs/app_spec.md
        Decomposes into feature specs: specs/features/user-auth.md,
                                       specs/features/recipe-crud.md, etc.

        [Phase 2 — STORIES]
        Breaks each feature into user stories:
        US-001: User registration with email verification
        US-002: User login with JWT tokens
        ...
        Dependency graph: US-001 → US-002 → US-003
        Parallel groups: Group A (US-001, US-004), Group B (US-002, US-005)

        [Phase 3-5 — DESIGN, TEST PLAN, EXEC PLAN]
        Produces design doc, test plan (with TC-001, TC-002...), execution plan.

        ──────────────────────────────────────────
        CHECKPOINT 1: Here's the plan. Approve?
        ──────────────────────────────────────────

You:    Looks good, go ahead.

Forge:  [Phase 6 — IMPLEMENT]
        Detects: 6 stories, 3 parallel groups → activates agent teams
        TeamCreate → TaskCreate (6 tasks) → spawns 3 implementer agents
        Each agent: TDD per criterion → validation → commit → next task

        [Phase 7-9 — TEST FILL, E2E, DEVOPS]
        Fills coverage gaps, creates E2E tests, generates CI/CD config.

        [Phase 10 — REVIEW]
        8 reviewers run in parallel. All 5 gating reviewers approve.

        ──────────────────────────────────────────
        CHECKPOINT 2: Review passed. Create PR?
        ──────────────────────────────────────────

You:    Yes, create the PR.

Forge:  [Phase 11 — PR]
        Creates PR with story-based commits and review summary.
```

### Add a Feature to an Existing App

```
You:    /add-feature add real-time notifications via WebSocket

Forge:  Same pipeline, but starts with a feature interview (not app spec).
        Reads existing specs/app_spec.md for context.
        Produces specs/features/notifications.md, then stories, design, etc.
```

### Quick Build Without Spec Ceremony

```
You:    /just-do-it add a health check endpoint at GET /health

Forge:  1. Parses your request into a structured brief
        2. Implements with TDD (RED-GREEN-REFACTOR)
        3. Runs 3-checker validation (architecture, design, PRD)
        4. Runs full test suite + linters
        5. Commits
```

### Resume an Interrupted Pipeline

```
You:    /resume

Forge:  Reads specs/pipeline_status.md:
        Phase 1-5: DONE
        Phase 6: IN PROGRESS
        → Resumes implementation from where it stopped
```

### Debug a Problem

```
You:    /debug users are getting 500 errors on the login endpoint

Forge:  5-phase debugging:
        1. Reproduce — find the failing request/test
        2. Isolate — narrow down to the failing component
        3. Hypothesize — form a theory about the root cause
        4. Fix — make the minimal change
        5. Verify — run tests, confirm the fix
```

### Review Code Quality

```
You:    /review

Forge:  Runs 8 reviewers in parallel:
        - 5 gating (spec, code, security, performance, architecture)
        - 3 advisory (design, simplifier, PRD-architecture)
        Produces specs/review_report.md with verdicts and findings.
```

### Reverse-Engineer Docs from Code

```
You:    /source-architecture

Forge:  Analyzes src/ directory structure, imports, and patterns.
        Generates architecture docs describing the current state of the code.
```

---

## Project Structure

```
claude-code-forge/
├── .claude/
│   ├── .claude-plugin/
│   │   └── plugin.json              # Plugin manifest (v2.0.0)
│   ├── .mcp.json                    # MCP server configuration
│   ├── settings.json                # Permissions + agent teams env var
│   ├── agents/                      # 20 agent definitions (markdown)
│   │   ├── spec-writer.md
│   │   ├── implementer.md
│   │   ├── task-executor.md
│   │   ├── test-writer.md
│   │   ├── e2e-writer.md
│   │   ├── devops.md
│   │   ├── spec-reviewer.md
│   │   ├── code-reviewer.md
│   │   ├── security-reviewer.md
│   │   ├── performance-reviewer.md
│   │   ├── architecture-alignment-checker.md
│   │   ├── design-consistency-checker.md
│   │   ├── code-simplifier.md
│   │   ├── prd-architecture-checker.md
│   │   ├── housekeeper.md
│   │   ├── pr-writer.md
│   │   ├── refactorer.md
│   │   ├── research-agent.md
│   │   ├── debug-agent.md
│   │   └── pipeline-orchestrator.md
│   ├── commands/                    # 23 slash commands
│   │   ├── build.md
│   │   ├── add-feature.md
│   │   ├── just-do-it.md
│   │   ├── build-unplanned.md
│   │   ├── resume.md
│   │   ├── init.md
│   │   ├── design.md
│   │   ├── design-product.md
│   │   ├── design-architecture.md
│   │   ├── work-on.md
│   │   ├── work-on-next.md
│   │   ├── create-tasks.md
│   │   ├── review.md
│   │   ├── validate.md
│   │   ├── check.md
│   │   ├── source-architecture.md
│   │   ├── source-specs.md
│   │   ├── spec-sync.md
│   │   ├── debug.md
│   │   ├── refactor.md
│   │   ├── create-pr.md
│   │   ├── skills.md
│   │   └── help.md
│   ├── skills/                      # 23 skills (each has SKILL.md)
│   │   ├── routing/
│   │   ├── sdlc-pipeline/
│   │   ├── layer-enforcement/
│   │   ├── conventions/
│   │   ├── test-driven-development/
│   │   ├── understanding-feature-requests/
│   │   ├── teams/
│   │   ├── validation-loop/
│   │   ├── task-validation-loop/
│   │   ├── implement-feature/
│   │   ├── execute-task/
│   │   ├── next-task/
│   │   ├── tasks/
│   │   ├── build-unplanned-feature/
│   │   ├── verification-suite/
│   │   ├── spec-sync-engine/
│   │   ├── worktree-isolation/
│   │   ├── architecture/
│   │   ├── design-system/
│   │   ├── check-alignment/
│   │   ├── sync-architecture/
│   │   ├── sync-design-system/
│   │   └── debugging/
│   ├── hooks/                       # Hook registrations + 11 scripts
│   │   ├── hooks.json               # Event → script mappings
│   │   └── scripts/
│   │       ├── pre_write_check.py
│   │       ├── post_write_lint.py
│   │       ├── pre_read_scaffold_guard.py
│   │       ├── post_commit_spec_check.py
│   │       ├── session_start.py
│   │       ├── pre_compact_save.py
│   │       ├── commit_alignment.py
│   │       ├── subagent_validate.py
│   │       ├── team_activity_log.py
│   │       ├── teammate_idle_check.py
│   │       └── task_completed_check.py
│   ├── linters/                     # Custom linters (portable)
│   │   ├── lint_all.py              # Runs all linters
│   │   ├── layer_deps.py            # Layer dependency checker
│   │   └── file_size.py             # File/function size limits
│   ├── docs/                        # Internal reference docs
│   │   ├── architecture.md          # 6-layer model reference
│   │   ├── pipeline.md              # Pipeline runbook
│   │   ├── conventions.md           # Coding conventions
│   │   ├── testing-standard.md      # Testing rules and standards
│   │   └── git-workflow.md          # Branch strategy and commit conventions
│   ├── templates/                   # Spec + project templates
│   │   ├── app_spec.md
│   │   ├── feature_spec.md
│   │   ├── feature_spec_lite.md
│   │   ├── user_stories.md
│   │   ├── design_doc.md
│   │   ├── test_plan.md
│   │   ├── execution_plan.md
│   │   ├── pipeline_status.md
│   │   ├── project/                 # Jinja2 scaffolding templates
│   │   │   ├── CLAUDE.md.j2
│   │   │   ├── pyproject.toml.j2
│   │   │   ├── Makefile.j2
│   │   │   ├── Dockerfile.j2
│   │   │   └── ci.yml.j2
│   │   └── e2e/
│   │       └── conftest.py
│   ├── evals/                       # Reviewer calibration samples
│   └── scripts/                     # Utility scripts
│       ├── validate_plugin.py       # Plugin integrity checker (104 checks)
│       ├── scaffold.py              # Project scaffolding
│       └── migrate_from_v2.py       # Migration from scaffold v2
├── CLAUDE.md                        # Project instructions for Claude Code
└── README.md                        # This file
```

---

## Configuration

### `.claude/settings.json`

Controls permissions and environment variables:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  },
  "permissions": {
    "allow": [
      "Bash(python3 *)", "Bash(pytest *)", "Bash(git *)",
      "Bash(gh *)", "Bash(docker *)", "Bash(make *)", ...
    ]
  }
}
```

- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` enables agent teams (experimental feature)
- The `permissions.allow` list controls which bash commands agents can run without prompting
- Teammates inherit these permissions at spawn time

**User-level settings** (in `~/.claude/settings.json`, not the project):

```json
{
  "teammateMode": "auto"
}
```

- `teammateMode` controls how agent team teammates are displayed: `"auto"` (default), `"in-process"`, or `"tmux"`
- Can also be set per-session with `claude --teammate-mode <mode>`

### Quality Gate Thresholds

| Gate | Threshold | Configured In |
|------|-----------|---------------|
| Test coverage | 80% minimum | `pytest --cov-fail-under=80` |
| File size | 300 lines max | `.claude/linters/file_size.py` |
| Function size | 50 lines max | `.claude/linters/file_size.py` |
| Gating reviewers | 5/5 unanimous | `.claude/skills/validation-loop/SKILL.md` |
| Retry cycles | 3 max | `.claude/skills/validation-loop/SKILL.md` |
| Team threshold | 4+ stories AND 2+ parallel groups | `.claude/docs/pipeline.md` |

### Coding Conventions

| Convention | Rule |
|-----------|------|
| File names | `snake_case.py` |
| Classes | `PascalCase` |
| Constants | `UPPER_SNAKE_CASE` |
| Type hints | Required on all functions (including tests) |
| Logging | `logging.getLogger(__name__)` — never `print()` |
| Error handling | Catch specific exceptions in Repo layer, log with context |
| Imports | stdlib, then third-party, then project (in layer order) |
| Domain types | Refined Pydantic types (`UserId`, `Email`), never raw `str`/`int` |
| Git branches | `feature/<story-id>-<brief-name>` |
| Commits | `feat(US-XXX): <description>` (one commit per story) |

---

## Extending Forge

### Adding a Custom Agent

Create a markdown file in `.claude/agents/`:

```markdown
# My Custom Agent

## Role

<What this agent does — one paragraph>

## Process

<Step-by-step instructions>

## Rules

- <Constraint 1>
- <Constraint 2>

## Allowed Tools

- **Read**, **Write**, **Edit**, **Bash**, **Glob**, **Grep**

## Output

<What the agent produces>
```

Agents use **no YAML frontmatter** — they are plain markdown files.

### Adding a Custom Skill

Create a directory in `.claude/skills/<skill-name>/` with a `SKILL.md` file:

```markdown
---
description: "Short description of what this skill does"
disable-model-invocation: true
---

# My Custom Skill

<Skill instructions>
```

- `description` in frontmatter is required
- `disable-model-invocation: true` means the skill is manually invoked only
- Omit `disable-model-invocation` (or set to `false`) for auto-invoked skills

### Adding a Custom Command

Create a markdown file in `.claude/commands/`:

```markdown
---
disable-model-invocation: true
---

<Command instructions that reference a skill or provide direct behavior>
```

Commands always have `disable-model-invocation: true` in frontmatter.

### Adding a Custom Hook

1. Create a Python script in `.claude/hooks/scripts/`
2. Add an entry to `.claude/hooks/hooks.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "python3 .claude/hooks/scripts/my_hook.py",
            "timeout": 10000
          }
        ]
      }
    ]
  }
}
```

Hook exit codes:
- `0` — allow the action
- `2` — reject with feedback (the hook's stderr is sent back as guidance)

Available events: `PreToolUse`, `PostToolUse`, `Stop`, `TeammateIdle`, `TaskCompleted`

---

## Troubleshooting

### Pipeline is stuck / no progress

```
/resume
```

This reads `specs/pipeline_status.md` and continues from the last completed phase. If the file doesn't exist, start fresh with `/build`.

### Linter failures after implementation

```bash
python3 .claude/linters/lint_all.py
```

Common issues:
- **Layer dependency violation**: a file in `src/service/` imports from `src/runtime/`. Move the import target to a lower layer or restructure.
- **File too large**: split files exceeding 300 lines into focused modules.

### Tests failing

```bash
pytest tests/ -x -q --tb=short
```

The `-x` flag stops at the first failure. Read the traceback and fix the issue.

### Coverage below 80%

```bash
pytest tests/ --cov=src --cov-report=term-missing
```

The `term-missing` report shows exactly which lines are uncovered. Add tests for those paths.

### `/scaffold` doesn't create any files in a new project

If `/scaffold` fails silently, tell Claude the absolute path to your forge clone so it can find the scaffold script:

```
Run: python3 ~/claude-code-forge/.claude/scripts/scaffold_project.py python-fastapi
```

After scaffolding, the full `.claude/` directory is copied into your project. You can then restart Claude without `--plugin-dir`.

> **Why `/scaffold` and not `/init`?** Claude Code has a built-in `/init` command that creates a basic `CLAUDE.md`. The forge uses `/scaffold` to avoid conflicting with it. When bootstrapping via `--plugin-dir`, use the namespaced version: `/claude-code-forge:scaffold`.

### Agent teams not activating

Verify the env var is set in `.claude/settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Also verify the stories file has 4+ stories AND 2+ parallel groups — both conditions must be met.

### Teammates not appearing

- In in-process mode, teammates may already be running but not visible. Press `Shift+Down` to cycle through them.
- If you requested split panes, ensure tmux is installed: `which tmux`
- For iTerm2, verify the `it2` CLI is installed and the Python API is enabled in iTerm2 preferences.

### Teammate stuck or unresponsive

In `in-process` display mode, press `Shift+Down` to cycle to the stuck teammate and message it directly. In `tmux` mode, click on the teammate's pane. Options:

- Give the teammate additional instructions directly
- Spawn a replacement teammate to continue the work

### Lead implementing instead of delegating

Sometimes the lead starts implementing tasks itself instead of waiting for teammates. Tell it:

```
Wait for your teammates to complete their tasks before proceeding.
```

### Task status appears stuck

Teammates sometimes fail to mark tasks as completed, which blocks dependent tasks. If a task appears stuck:
1. Check whether the work is actually done
2. Update the task status manually via TaskUpdate or tell the lead to nudge the teammate

### Teammates lost after session resume

`/resume` and `/rewind` do not restore in-process teammates. After resuming a session, the lead may attempt to message teammates that no longer exist. Tell the lead to spawn new teammates.

### Orphaned tmux sessions

If a tmux session persists after the team ends, list and kill it manually:

```bash
tmux ls
tmux kill-session -t <session-name>
```

### Too many permission prompts from teammates

Teammate permission requests bubble up to the lead. Pre-approve common operations in your permission settings (`.claude/settings.json` `permissions.allow`) before spawning teammates to reduce interruptions.

### Review keeps failing after 3 cycles

After 3 retry cycles, forge escalates to you. Options:
1. Fix the remaining issues manually
2. Override by telling forge to proceed
3. Run `/review` again after making changes

### Plugin validation fails

```bash
python3 .claude/scripts/validate_plugin.py
```

This checks all 104 expected files. Missing files will be listed with `[MISSING]` status.

---

## Development

### Validate Plugin Integrity

```bash
python3 .claude/scripts/validate_plugin.py
# Expected: 104/104 PASS
```

### Test with a Fresh Project

```bash
mkdir test-project && cd test-project && git init
claude --plugin-dir ~/claude-code-forge/.claude
> /claude-code-forge:scaffold python-fastapi
> /exit
claude
> /build a simple todo API
```

### Migration from Scaffold v2

```bash
python3 .claude/scripts/migrate_from_v2.py /path/to/your/project
```

This removes plugin-managed files from the project directory (agents, hooks, templates) while preserving your `specs/` and linters.
