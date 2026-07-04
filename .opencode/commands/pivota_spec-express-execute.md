---
description: Express execution phase — executes the PLAN.md files created by /pivota_spec-express-plan, wave-by-wave, with integration contract gates between waves. Aggregates summaries, updates STATE.md, and runs UAT.
argument-hint: "[slug from express-plan]"
tools:
  read: true
  write: true
  edit: true
  glob: true
  grep: true
  bash: true
  task: true
  question: true
---
<objective>
Execute an Express plan previously created by /pivota_spec-express-plan.

Reads `.planning/express/${slug}/` for the WAVE-SCHEDULE.md and PLAN.md files,
executes plans wave-by-wave (fresh executor per plan, parallel within a wave),
verifies integration contracts at each wave boundary, aggregates summaries,
updates STATE.md, and runs UAT verification.

**Prerequisite:** `/pivota_spec-express-plan` must have run and plans must be approved.

**Usage:**
- `/pivota_spec-express-execute my-app-slug` — execute a specific plan set
- `/pivota_spec-express-execute` — auto-resolves to the most recently created express plan
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/express-execute.md
</execution_context>

<context>
@.planning/STATE.md
</context>

<process>
Execute the express-execute workflow from @/home/daytona/.config/opencode/pivota_spec-framework/workflows/express-execute.md end-to-end.

Express task slug (used to locate .planning/express/${slug}/):

$ARGUMENTS
</process>

