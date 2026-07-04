---
description: Security-audit a completed phase — verify threat mitigations or build a STRIDE register from the diff, then produce SECURITY.md
argument-hint: "[phase number, e.g. '\''4'\''] | --express <slug> [--diff-base <ref>]"
tools:
  read: true
  bash: true
  glob: true
  grep: true
  write: true
  task: true
---
<objective>
Retroactively security-audit a completed phase'\''s implemented code and produce a SECURITY.md report.

Purpose: confirm the phase didn'\''t ship exploitable issues. If the phase PLAN.md carries a `<threat_model>`, verify each declared mitigation exists in code; otherwise build a STRIDE register from the diff and audit it. Findings are adversarially refuted to keep false positives low.

Output: {phase}-SECURITY.md with a verdict (SECURED / OPEN_THREATS), audited attack surface, and confirmed findings. Under `block` enforcement, open HIGH/CRITICAL findings gate phase advancement.
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/secure-phase.md
@/home/daytona/.config/opencode/pivota_spec-framework/templates/SECURITY.md
</execution_context>

<context>
Phase: $ARGUMENTS (optional)
- If provided: audit that phase (e.g., "4")
- If not provided: resolve the most recently executed phase from STATE.md
- If `--express <slug>` is passed: the audit is **whole-diff** (no phase resolution). The STRIDE register is built retroactively over the entire diff (`git diff <diff-base>...HEAD`, default base `main`), and SECURITY.md is written to `.planning/express/<slug>/SECURITY.md` (the express slug dir). Optional `--diff-base <ref>` overrides the diff base.

@.planning/STATE.md
@.planning/ROADMAP.md
@.planning/config.json
</context>

<process>
Execute the secure-phase workflow from @/home/daytona/.config/opencode/pivota_spec-framework/workflows/secure-phase.md end-to-end.
If invoked with `--express <slug>`, run the workflow'\''s whole-diff express branch (skip the A/B/C phase resolution; write SECURITY.md to `.planning/express/<slug>/SECURITY.md`); otherwise resolve a phase as today.
Preserve all gates: input-state resolution (A/B/C in phase mode, or the EXPRESS branch in express mode), the read-only auditor agent (pivota_spec-security-auditor), SECURITY.md generation, and the enforcement policy (off / warn / block). Never patch implementation files from this command — report findings; fixes go through execute-phase.
</process>

