---
description: Execute a quick task with Pivota Spec guarantees (atomic commits, state tracking) but skip optional agents
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
Execute small, ad-hoc tasks with Pivota Spec guarantees (atomic commits, STATE.md tracking) while skipping optional agents (research, plan-checker, verifier).

Quick mode is the same system with a shorter path:
- Spawns pivota_spec-planner (quick mode) + pivota_spec-executor(s)
- Skips pivota_spec-phase-researcher, pivota_spec-plan-checker, pivota_spec-verifier
- Quick tasks live in `.planning/quick/` separate from planned phases
- Updates STATE.md "Quick Tasks Completed" table (NOT ROADMAP.md)

Use when: You know exactly what to do and the task is small enough to not need research or verification.
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/quick.md
</execution_context>

<context>
@.planning/STATE.md
</context>

<process>
Execute the quick workflow from @/home/daytona/.config/opencode/pivota_spec-framework/workflows/quick.md end-to-end.
Preserve all workflow gates (validation, task description, planning, execution, state updates, commits).

User'\''s task description (may be empty — workflow Step 1 will prompt if so):

$ARGUMENTS
</process>

