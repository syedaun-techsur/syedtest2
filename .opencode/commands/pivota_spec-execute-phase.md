---
description: Execute all plans in a phase with wave-based parallelization
argument-hint: "<phase-number> [--gaps-only] [--no-transition]"
tools:
  read: true
  write: true
  edit: true
  glob: true
  grep: true
  bash: true
  task: true
  todowrite: true
  question: true
---
<objective>
Execute all plans in a phase using wave-based parallel execution.

Orchestrator stays lean: discover plans, analyze dependencies, group into waves, spawn subagents, collect results. Each subagent loads the full execute-plan context and handles its own plan.

Context budget: ~15% orchestrator, 100% fresh per subagent.
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/execute-phase.md
@/home/daytona/.config/opencode/pivota_spec-framework/references/ui-brand.md
</execution_context>

<context>
Phase: $ARGUMENTS

**Flags:**
- `--gaps-only` — Execute only gap closure plans (plans with `gap_closure: true` in frontmatter). Use after verify-work creates fix plans.
- `--no-transition` — After execution and verification, return completion status and STOP. Do not offer next phase or auto-advance. Used when spawned by an external orchestrator (e.g., Pivota card execution).

@.planning/ROADMAP.md
@.planning/STATE.md
</context>

<process>
Execute the execute-phase workflow from @/home/daytona/.config/opencode/pivota_spec-framework/workflows/execute-phase.md end-to-end.
Preserve all workflow gates (wave execution, checkpoint handling, verification, state updates, routing).
</process>

