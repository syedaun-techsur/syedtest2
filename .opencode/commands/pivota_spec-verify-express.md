---
description: Express verification phase — automated UAT for a completed express task. Builds the project, starts the app, generates and runs Playwright end-to-end tests from acceptance criteria, drives an auto-fix loop, and records results in UAT.md and STATE.md without requiring user input.
argument-hint: "[slug from express-execute]"
tools:
  read: true
  write: true
  edit: true
  glob: true
  grep: true
  bash: true
  task: true
---
<objective>
Run automated UAT verification for an Express task previously built by
/pivota_spec-express-execute.

Builds the project, starts the application, generates Playwright end-to-end
tests from the task'\''s acceptance criteria, executes them, and drives an
auto-fix goal loop until all tests pass or the retry cap is reached. Records
all results in `.planning/express/${slug}/UAT.md` and updates STATE.md — no
user input is required during the run.

**Prerequisite:** `/pivota_spec-express-execute` must have completed and
produced `.planning/express/${slug}/SUMMARY.md`.

**Usage:**
- `/pivota_spec-verify-express my-app-slug` — verify a specific express task
- `/pivota_spec-verify-express` — auto-resolves to the most recently completed express task
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/verify-express.md
</execution_context>

<context>
@.planning/STATE.md
</context>

<process>
Execute the verify-express workflow from @/home/daytona/.config/opencode/pivota_spec-framework/workflows/verify-express.md end-to-end.
Preserve all workflow gates (build detection, app startup, test generation, auto-fix loop, UAT.md + STATE.md updates).

Express task slug (used to locate .planning/express/${slug}/):

$ARGUMENTS
</process>

