---
description: Execute the roadmapper agent to create or regenerate project roadmap
argument-hint: "[--force]"
tools:
  read: true
  bash: true
  write: true
  task: true
  question: true
---
<context>
**Flags:**
- `--force` — Skip confirmation prompt. Automatically regenerate roadmap when files exist.
</context>

<objective>
Execute the pivota_spec-roadmapper agent to create or regenerate the project roadmap (ROADMAP.md, STATE.md) from existing PROJECT.md and REQUIREMENTS.md.

**Creates/Updates:**
- `.planning/ROADMAP.md` — phase structure with success criteria
- `.planning/STATE.md` — project memory and current position
- `.planning/REQUIREMENTS.md` — updates traceability section

**Prerequisites:**
- `.planning/PROJECT.md` must exist
- `.planning/REQUIREMENTS.md` must exist

**After this command:** Run `/pivota_spec-plan-phase 1` to start execution.
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/roadmapper.md
@/home/daytona/.config/opencode/agents/pivota_spec-roadmapper.md
</execution_context>

<process>
Execute the roadmapper workflow from @/home/daytona/.config/opencode/pivota_spec-framework/workflows/roadmapper.md end-to-end.
Preserve all workflow gates (validation, approvals, commits, routing).
</process>

