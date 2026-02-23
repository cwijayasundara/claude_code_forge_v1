# claude-code-forge

> Humans are harness engineers. Agents write all code. Specs are the source of truth.

## Request Routing (CRITICAL)

Before writing any code, classify the user's request:

| Pattern | Category | Action |
|---------|----------|--------|
| "Build me X", "Create a", "Add feature X", "New app" | **Pipeline** | Invoke `forge:sdlc-pipeline` — do NOT explore or plan first |
| "Just do it", "Quick change", "Skip the spec" | **Quick Build** | Invoke `forge:build-unplanned-feature` — TDD without spec ceremony |
| "Continue", "Resume", "What's next" | **Resume** | Read `specs/pipeline_status.md`, resume from next incomplete phase |
| Everything else (bug fixes, refactoring, questions) | **Toolbox** | Handle directly; `/forge:*` commands available |

**Never skip the spec-writer for "build" or "add feature" requests.** This is mandatory routing, not optional.

## Architecture: 6-Layer Model

```
Types(0) → Config(1) → Repo(2) → Service(3) → Runtime(4) → UI(5)
```

Code may only depend **forward** through layers. Backward imports are forbidden and enforced by `.claude/linters/layer_deps.py`.

| Layer | Path | May Import From | Never Import From |
|-------|------|-----------------|-------------------|
| Types | `src/types/` | (none) | All others |
| Config | `src/config/` | Types | Repo, Service, Runtime, UI |
| Repo | `src/repo/` | Types, Config | Service, Runtime, UI |
| Service | `src/service/` | Types, Config, Repo | Runtime, UI |
| Runtime | `src/runtime/` | Types, Config, Repo, Service | UI |
| UI | `src/ui/` | All layers | (none) |

Key constraints:
- **Service has no direct I/O** — all DB, HTTP, filesystem access goes through Repo
- **Types has no project imports** — standard library and Pydantic only
- **No raw primitives for domain concepts** — use refined Pydantic types (`UserId`, `Email`)
- **No global mutable state** — config loaded once in Config, injected forward

## SDLC Pipeline (11 Phases)

```
SPEC → STORIES → DESIGN → TEST PLAN → EXEC PLAN → [APPROVE]
→ IMPLEMENT → TEST FILL → E2E → DEVOPS → REVIEW → [APPROVE] → PR
```

Two human approval checkpoints: before implementation (phase 6) and before PR (phase 11).

Update `specs/pipeline_status.md` after each phase completes.

## TDD Enforcement (Iron Law)

Every acceptance criterion follows the TDD cycle — this is mandatory, not optional:

1. **RED** — Write a failing test that asserts the expected behavior
2. **GREEN** — Write the minimum production code to pass
3. **REFACTOR** — Clean up without changing behavior

Never write production code without a failing test first. See `test-driven-development` skill.

## Validation Loops

### Full Validation (Phase 10 — 8 agents, parallel)

5 gating reviewers (must all APPROVE unanimously) + 3 advisory reviewers:

| # | Agent | Type |
|---|-------|------|
| 1 | spec-reviewer | Gating |
| 2 | code-reviewer | Gating |
| 3 | security-reviewer | Gating |
| 4 | performance-reviewer | Gating |
| 5 | architecture-alignment-checker | Gating |
| 6 | design-consistency-checker | Advisory |
| 7 | code-simplifier | Advisory |
| 8 | prd-architecture-checker | Advisory |

Max 3 retry cycles. See `validation-loop` skill.

### Per-Task Validation (after each story — 3 checkers)

| # | Agent |
|---|-------|
| 1 | architecture-alignment-checker |
| 2 | design-consistency-checker |
| 3 | prd-architecture-checker |

See `task-validation-loop` skill.

## Quality Gates

- **Linters**: `python3 .claude/linters/lint_all.py` (layer deps + file size)
- **Tests**: `pytest tests/ --cov=src --cov-fail-under=80`
- **File limits**: 300 lines per file, 50 lines per function
- **Review**: 5 gating reviewers + 3 advisory reviewers (all gating must pass)

## Conventions

- **Files**: `snake_case.py` | **Classes**: `PascalCase` | **Constants**: `UPPER_SNAKE_CASE`
- **Logging**: `logging.getLogger(__name__)` — never `print()`
- **Error handling**: catch specific exceptions in Repo layer, log with context
- **Type hints**: all function signatures, no `Any`
- **Imports**: stdlib → third-party → project (in layer order)
- **Git**: feature branches `feature/<story-id>-<brief-name>`, one commit per story

## Agents (20)

| Agent | Role |
|-------|------|
| spec-writer | Socratic interviewer → specs, stories, design, test plan, exec plan |
| implementer | Code + tests from spec, story-by-story, TDD enforced |
| task-executor | TDD-enforced individual task execution with post-task validation |
| test-writer | Coverage gap filler (target 80%+) |
| e2e-writer | Playwright E2E + API contract tests |
| devops | CI/CD pipelines, Dockerfiles, infrastructure (default: Azure) |
| spec-reviewer | Validates implementation matches spec |
| code-reviewer | Code quality, conventions, performance |
| security-reviewer | OWASP Top 10, auth, secrets |
| performance-reviewer | N+1 queries, unbounded loops, missing pagination, caching |
| architecture-alignment-checker | Code-vs-architecture-docs alignment |
| design-consistency-checker | UI/UX compliance against design system |
| code-simplifier | Over-engineering detection (advisory) |
| prd-architecture-checker | Architecture-vs-product-requirements alignment |
| housekeeper | Dead code, unused imports, stale TODOs cleanup |
| pr-writer | Structured PRs with story-based commits |
| refactorer | Technical debt reduction, structure only |
| research-agent | Read-only codebase analysis |
| debug-agent | Systematic 5-phase debugging |
| pipeline-orchestrator | SDLC phase transitions |

## Team Orchestration (Claude Code Agent Teams)

> Experimental. Enabled via `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in `.claude/settings.json`.

**MUST** use teams when: 4+ stories AND 2+ parallel groups. This is a mechanical check — do not override based on subjective judgment about story size or complexity.

Each teammate is a **full, independent Claude Code session** with its own context window. Teammates coordinate through a shared task list and direct messaging.

Team lifecycle (managed by `teams` skill):
- **TeamCreate** — create team `feat-<feature-name>` (shared task list at `~/.claude/tasks/`)
- **TaskCreate** — one task per story with dependency blocking and file ownership
- **Task** — spawn teammates (one per parallel group), each a full Claude Code session
- **SendMessage** — teammates report completion, message each other directly
- **TaskList/TaskGet/TaskUpdate** — self-claim tasks, track progress, auto-unblock dependencies
- **shutdown_request** — graceful teammate shutdown before cleanup
- **TeamDelete** — cleanup (only the lead runs this)

Quality gate hooks:
- **TeammateIdle** — checks for incomplete tasks before allowing idle
- **TaskCompleted** — runs linters and tests before allowing task completion

## Project Structure

```
.claude/
├── agents/           20 specialized agents
├── commands/         23 slash commands (/forge:*)
├── docs/             Architecture, conventions, pipeline, testing, git workflow
├── evals/            Reviewer calibration samples
├── hooks/            Pre/post tool-use hooks + 11 scripts (incl. TeammateIdle, TaskCompleted)
├── linters/          layer_deps.py, file_size.py, lint_all.py
├── scripts/          scaffold, validate, migrate utilities
├── settings.json     Default permissions + agent teams enabled
├── skills/           23 auto/manual skills
└── templates/        Spec templates + project scaffolding (Jinja2)
```
