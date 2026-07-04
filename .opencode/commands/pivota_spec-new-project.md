---
description: Initialize a new project with deep context gathering and PROJECT.md
argument-hint: "[project description] [--auto]"
tools:
  read: true
  bash: true
  write: true
  task: true
  question: true
---
<context>
**Flags:**
- `--auto` — Automatic mode. After config questions, runs research → requirements → roadmap without further interaction. Expects idea document via @ reference.
- `--spec-express` — Spec-Express mode. After writing PROJECT.md + config.json + STATE.md, chains directly into `/pivota_spec-express-plan` (inline — do NOT spawn a new command). Skips REQUIREMENTS.md and ROADMAP.md. Sets `workflow_mode: "spec-express"` in `.planning/config.json`. Subsequent execution uses `/pivota_spec-express-execute`.
- `--skip-config` — Workflow config was already written by the frontend (`.planning/config.json` exists). Skip the config questioning round.
</context>

<objective>
Initialize a new project through unified flow: questioning → research (optional) → requirements → roadmap.

**Creates:**
- `.planning/PROJECT.md` — project context
- `.planning/config.json` — workflow preferences
- `.planning/research/` — domain research (optional)
- `.planning/REQUIREMENTS.md` — scoped requirements
- `.planning/ROADMAP.md` — phase structure
- `.planning/STATE.md` — project memory
- `project_specs/` — spec documemnt repo

**After this command (phase mode):** Run `/pivota_spec-plan-phase 1` to start execution.
**In spec-express mode:** Chains automatically into `/pivota_spec-express-plan` — no separate command needed.
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/new-project.md
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/express-plan.md
@/home/daytona/.config/opencode/pivota_spec-framework/references/questioning.md
@/home/daytona/.config/opencode/pivota_spec-framework/references/ui-brand.md
@/home/daytona/.config/opencode/pivota_spec-framework/templates/project.md
@/home/daytona/.config/opencode/pivota_spec-framework/templates/requirements.md
</execution_context>

<project_description>
$ARGUMENTS
</project_description>

<process>
YOU are the orchestrator. Follow the workflow steps from @/home/daytona/.config/opencode/pivota_spec-framework/workflows/new-project.md DIRECTLY — do NOT re-invoke this command or spawn tasks to run it again.

The `<project_description>` section above contains the user'\''s input. It may include:
- `--skip-config` flag: means `.planning/config.json` was already written. Skip config questions in Step 5.
- `--spec-express` flag: spec-express mode. After Step 4 (PROJECT.md) and Step 5 (config.json with `workflow_mode: "spec-express"`), write a minimal STATE.md, then **inline-execute the express-plan workflow** (`@/home/daytona/.config/opencode/pivota_spec-framework/workflows/express-plan.md`) using the project description as the task. Do NOT run Steps 6/7/8 (research, requirements, roadmap). Do NOT stop — chain directly into express-plan.
- **Project description text**: If there is descriptive text (beyond flags), the user already told you what they want to build. Skip Step 3 (Deep Questioning) and use the text directly for Step 4 (Write PROJECT.md).

Execute the workflow steps in order. Preserve all workflow gates (validation, approvals, commits, routing).
</process>

