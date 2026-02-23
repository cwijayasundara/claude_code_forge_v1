---
disable-model-invocation: true
---

# /forge:init

Scaffold a new project with the 6-layer architecture, specs directories, linters, and configuration files.

## When to Use

Use this command to initialize a new project from scratch. This sets up the full directory structure, configuration files, and quality gates so you can immediately begin building with `/forge:build`.

## Arguments

- **project-type** (required): The project template to use. Supported values:
  - `python-fastapi` — Python + FastAPI backend
  - `python-django` — Python + Django backend
  - `node-express` — Node.js + Express backend
  - Additional types may be available in `.claude/scripts/scaffold_project.py`

Example: `/forge:init python-fastapi`

## Process

1. **Parse the project type** from the user's argument. If no argument is provided, ask the user which project type they want.

2. **Run the scaffold script**:
   ```
   python3 .claude/scripts/scaffold_project.py <project-type>
   ```

3. **Verify the scaffold created these directories**:
   - `src/types/`, `src/config/`, `src/repo/`, `src/service/`, `src/runtime/`, `src/ui/` (6-layer architecture)
   - `tests/` with mirror structure matching `src/`
   - `specs/features/`, `specs/stories/`, `specs/design/`, `specs/tests/`, `specs/plans/`
   - `.claude/linters/` with `layer_deps.py` and `file_size.py`

4. **Verify configuration files exist**:
   - `CLAUDE.md` — project instructions (from template)
   - `pyproject.toml` or `package.json` — depending on project type
   - `Makefile` — with `test`, `lint`, `format` targets
   - `.env.example` — environment variable template

5. **Report results** to the user:
   - List all created directories and files
   - Suggest next step: "Run `/forge:build` to start building your application"

6. If the script fails or any expected artifacts are missing, report the error clearly and suggest manual fixes.
