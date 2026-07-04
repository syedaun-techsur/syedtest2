---
description: Conversational UAT verification for a quick task — walks the user through plan tasks + matched UserStories acceptance criteria, writes UAT.md, updates STATE.md, offers follow-up fix.
argument-hint: "[quick task id]"
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
Verify a completed quick task through conversational UAT.

**Inputs:** quick task ID (e.g., `260518-l1s`). If omitted, resolves to the most recent row in STATE.md'\''s Quick Tasks Completed table.

**Process:** Reads the task'\''s PLAN.md and any matching acceptance criteria from `project_specs/UserStories-*.md`, then walks the user through each criterion one at a time. Records pass/fail/skip into `${task_dir}/${id}-UAT.md` and updates STATE.md'\''s UAT column.

**Output:**
- `${task_dir}/${id}-UAT.md` — per-criterion verification record
- STATE.md UAT column updated for this row (`✓ M/N`, `⚠ K fails`, `— partial`, or `— skipped`)
- If failures exist: prompt user to spawn a follow-up `/pivota_spec-quick` fix
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/verify-quick.md
</execution_context>

<context>
Quick task ID: $ARGUMENTS

@.planning/STATE.md
</context>

<process>
Execute the verify-quick workflow from @/home/daytona/.config/opencode/pivota_spec-framework/workflows/verify-quick.md end-to-end.
Preserve all workflow gates (resolution, criteria derivation, conversational loop, UAT write, STATE update, commit, fail-loop prompt).
</process>

