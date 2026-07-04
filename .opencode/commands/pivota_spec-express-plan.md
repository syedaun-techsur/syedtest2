---
description: Express planning phase — initializes express directory, creates wave schedule and per-wave PLAN.md files from project_specs/. Ends with a review gate; run /pivota_spec-express-execute to build.
argument-hint: "[task description]"
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
Plan a full Express implementation from spec docs.

Reads `project_specs/` (PRD, TechArch, FRD, UserStories, etc.), produces a wave schedule,
then spawns one planner per wave domain. Each planner creates a focused PLAN.md with
integration contracts. Commits all plans and STOPS — the user reviews before executing.

**Output:**
- `.planning/express/${slug}/WAVE-SCHEDULE.md`
- `.planning/express/${slug}/01-PLAN.md`, `02-PLAN.md`, ... (one per wave)
- All committed and pushed

**Next step after review:** `/pivota_spec-express-execute ${slug}`
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/express-plan.md
</execution_context>

<context>
@.planning/STATE.md
</context>

<process>
Execute the express-plan workflow from @/home/daytona/.config/opencode/pivota_spec-framework/workflows/express-plan.md end-to-end.

User'\''s task description (may be empty — workflow Step 1 will prompt if so):

$ARGUMENTS
</process>

