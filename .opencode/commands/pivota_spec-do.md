---
description: Route freeform text to the right Pivota Spec command automatically
argument-hint: "<description of what you want to do>"
tools:
  read: true
  bash: true
  question: true
---
<objective>
Analyze freeform natural language input and dispatch to the most appropriate Pivota Spec command.

Acts as a smart dispatcher — never does the work itself. Matches intent to the best Pivota Spec command using routing rules, confirms the match, then hands off.

Use when you know what you want but don'\''t know which `/pivota_spec-*` command to run.
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec/workflows/do.md
@/home/daytona/.config/opencode/pivota_spec/references/ui-brand.md
</execution_context>

<context>
$ARGUMENTS
</context>

<process>
Execute the do workflow from @/home/daytona/.config/opencode/pivota_spec/workflows/do.md end-to-end.
Route user intent to the best Pivota Spec command and invoke it.
</process>

