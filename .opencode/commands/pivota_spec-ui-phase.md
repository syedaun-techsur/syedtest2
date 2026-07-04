---
description: Generate UI design contract (UI-SPEC.md) for frontend phases
argument-hint: "[phase]"
tools:
  read: true
  write: true
  bash: true
  glob: true
  grep: true
  task: true
  webfetch: true
  question: true
  mcp__context7__*: true
---
<objective>
Create a UI design contract (UI-SPEC.md) for a frontend phase.
Orchestrates pivota_spec-ui-researcher and pivota_spec-ui-checker.
Flow: Validate → Research UI → Verify UI-SPEC → Done
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec/workflows/ui-phase.md
@/home/daytona/.config/opencode/pivota_spec/references/ui-brand.md
</execution_context>

<context>
Phase number: $ARGUMENTS — optional, auto-detects next unplanned phase if omitted.
</context>

<process>
Execute @/home/daytona/.config/opencode/pivota_spec/workflows/ui-phase.md end-to-end.
Preserve all workflow gates.
</process>

