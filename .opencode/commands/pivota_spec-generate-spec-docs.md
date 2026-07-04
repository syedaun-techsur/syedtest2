---
description: Generate spec documentation (PRD, Personas, FRD, JTBD, TechArch, Journeys, UserStories, StoryMap, UX-Mockup) for a project. Pass a single compliance flag to generate only that document.
argument-hint: "[--prd-only | --skip-prd | --skip-personas | --skip-jtbd | --skip-journeys | --skip-storymap | --skip-userstories | --interactive | --regenerate | --fips | --business-case | --cost-benefit | --data-governance | --governance-docs | --skip-governance-docs]"
tools:
  read: true
  bash: true
  write: true
  task: true
  question: true
---
<context>
**Core Doc Flags:**
- `--prd-only` — Generate only PRD.md
- `--skip-prd` — Skip PRD generation (use existing)
- `--skip-personas` — Skip Personas generation
- `--skip-jtbd` — Skip JTBD generation (also skips Journeys and StoryMap)
- `--skip-journeys` — Skip Journeys generation (also skips StoryMap)
- `--skip-storymap` — Skip StoryMap generation
- `--skip-userstories` — Skip UserStories generation
- `--skip-ux` — Skip UX-Mockup generation
- `--skip-validation` — Skip spec validation
- `--interactive` — Use interactive mode for TechArch (collaborative planning with codebase analysis)
- `--regenerate` — Regenerate all documents from scratch

**Compliance & Governance Flags (standalone — skips all core doc generation):**
- `--fips` — Generate only FIPS Security Categorization
- `--business-case` — Generate only Business Case
- `--cost-benefit` — Generate only Cost-Benefit Analysis
- `--data-governance` — Generate only Data Governance Specification (auto-enables `--fips` if FIPS doc not already present)
- `--governance-docs` — Generate all 4 compliance docs without prompting
- `--skip-governance-docs` — Skip governance doc prompt when running full generation
</context>

<objective>
Generate professional spec documentation for the current project.

**Creates:**
- `project_specs/PRD.md` — Product Requirements Document
- `project_specs/PERSONAS-{PROJECT}.md` — User Persona Profiles
- `project_specs/FRD.md` — Functional Requirements Document
- `project_specs/JTBD-{PROJECT}.md` — Jobs to be Done
- `project_specs/TechArch.md` — Technical Architecture Document
- `project_specs/JOURNEYS-{PROJECT}.md` — User Journey Maps
- `project_specs/UserStories.md` — User Stories Document
- `project_specs/STORY-MAP-{PROJECT}.md` — Story Map
- `project_specs/UX-Mockup.md` — UX Mockup Document

**Document Dependencies:**
PRD → Personas → FRD → JTBD → TechArch → Journeys → UserStories → StoryMap → UX-Mockup → Validation

Each document builds on the previous one. Run after `/pivota_spec-new-project` completes.

**Interactive Mode (--interactive):**
TechArch uses collaborative planning with codebase analysis instead of auto-generation.
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/generate-spec-docs.md
</execution_context>

<process>
Execute the generate-spec-docs workflow from @/home/daytona/.config/opencode/pivota_spec-framework/workflows/generate-spec-docs.md end-to-end.

**Prerequisites:**
- `.planning/PROJECT.md` must exist (run `/pivota_spec-new-project` first)
- `project_specs/` directory will be created if it doesn'\''t exist

**Process:**
1. Initialize — Check prerequisites, create project_specs/ if needed
2. Gather requirements — Collect info from PROJECT.md, REQUIREMENTS.md, research/
3. Generate PRD — Spawn pivota_spec-prd-generator agent
4. Generate Personas — Spawn pivota_spec-personas-generator agent (depends on PRD)
5. Generate FRD — Spawn pivota_spec-frd-generator agent (depends on PRD)
6. Generate JTBD — Spawn pivota_spec-jtbd-generator agent (depends on PRD + Personas)
7. Generate TechArch — Spawn pivota_spec-techarch-generator agent (depends on PRD + FRD)
   - With --interactive: Uses collaborative planning mode
8. Generate Journeys — Spawn pivota_spec-journeys-generator agent (depends on Personas + JTBD)
9. Generate UserStories — Spawn pivota_spec-userstories-generator agent (depends on PRD + FRD)
10. Generate StoryMap — Spawn pivota_spec-storymap-generator agent (depends on Personas + JTBD + Journeys + UserStories)
11. Generate UX-Mockup — Spawn pivota_spec-ux-designer agent (depends on UserStories + Journeys)
12. Validate Specs — Spawn pivota_spec-validator agent (quality gate)
13. Report completion — Display summary table

Preserve all workflow gates (validation, commits).
</process>

