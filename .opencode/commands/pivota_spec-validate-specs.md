---
description: Validate spec documents for clarity and actionability. Use --techarch to stress-test architecture decisions.
argument-hint: "[--prd | --frd | --userstories | --techarch | --all | --skip-interview]"
tools:
  read: true
  bash: true
  write: true
  task: true
  question: true
---
<context>
**Flags:**
- `--prd` — Validate PRD.md only
- `--frd` — Validate FRD.md only
- `--userstories` — Validate UserStories.md only
- `--techarch` — Stress-test TechArch.md (architecture validation mode)
- `--all` — Validate all spec documents including TechArch
- `--skip-interview` — Report issues without asking clarifying questions
</context>

<objective>
Validate spec documents against quality dimensions:

**Specs Mode (default):**
- Problem definition clarity
- User experience requirements
- Functional requirements quality

**TechArch Mode (--techarch):**
- Stress-test critical architectural decisions
- 6 dimensions: Simplicity, Flexibility, Robustness, Scaling, Codebase Fit, Consistency
- Focus on the critical 30% that shapes 80-90% of implementation

**Process:**
1. Evaluate specs/architecture against validation dimensions
2. Identify gaps and ambiguities (or critical decisions for TechArch)
3. Interview user to resolve issues
4. Update original spec documents
5. Confirm readiness for implementation

**Output:** Updated spec docs with resolved issues.
</objective>

<execution_context>
@/home/daytona/.config/opencode/agents/pivota_spec-validator.md
</execution_context>

<process>
## 1. Prerequisites Check

**Verify spec documents exist:**
```bash
ls project_specs/PRD.md 2>/dev/null
ls project_specs/FRD.md 2>/dev/null
ls project_specs/UserStories.md 2>/dev/null
```

**If no spec docs found:**
```
Error: No spec documents found in project_specs/

Generate spec documents first:
/pivota_spec-generate-spec-docs
```

## 2. Determine Scope

**Parse flags from $ARGUMENTS:**
- `--prd` → Validate only PRD.md
- `--frd` → Validate only FRD.md
- `--userstories` → Validate only UserStories.md
- `--techarch` → Stress-test TechArch.md (architecture validation mode)
- `--all` → Validate all documents including TechArch
- `--skip-interview` → Report only, no clarifying questions

- Also check for TechArch.md
- Use TechArch validation mode for architecture stress-testing

## 3. Spawn Validator Agent

**If --techarch flag is set:**

Spawn validator in TechArch mode:

```
Task(
  subagent_type="pivota_spec-validator",
  prompt="
    Validate TechArch with stress-testing mode.
    
    <mode>
    Mode: --techarch
    Use TechArch validation mode from <techarch_validation_mode> section.
    </mode>
    
    <context>
    Read these files:
    - project_specs/TechArch.md
    - project_specs/PRD.md
    - project_specs/FRD.md
    - project_specs/UserStories.md
    - .planning/PROJECT.md
    </context>
    
    <validation_dimensions>
    1. Simplicity
    2. Flexibility
    3. Robustness
    4. Scaling
    5. Codebase Fit
    6. Consistency
    </validation_dimensions>
    
    [If --skip-interview NOT set:]
    Interview user to resolve issues.
    Update TechArch.md with clarifications.
  ",
  description="Validate TechArch with stress-testing"
)
```

**Otherwise (specs mode):**

Spawn the pivota_spec-validator agent with context:

Spawn the pivota_spec-validator agent with context:

```
Task(
  subagent_type="pivota_spec-validator",
  prompt="
    Validate spec documents for clarity, completeness, and actionability.
    
    <scope>
    [Based on flags: which documents to validate]
    </scope>
    
    <context>
    Read these files:
    - project_specs/PRD.md
    - project_specs/FRD.md
    - project_specs/UserStories.md
    - .planning/PROJECT.md (if available)
    </context>
    
    <validation_dimensions>
    1. Problem Definition & Context
       - Problem clarity
       - User context
       - Scope appropriateness
       - Success criteria
    
    2. User Experience Requirements
       - Flow documentation
       - Decision points
       - Edge cases
       - Error handling
    
    3. Functional Requirements Quality
       - Specificity
       - WHAT not HOW
       - Terminology consistency
       - Testability
    </validation_dimensions>
    
    <mode>
    [If --skip-interview:]
    Report issues only. Do not ask clarifying questions.
    
    [Otherwise:]
    For each issue found, ask clarifying questions.
    Update original documents with resolved issues.
    Confirm changes with user.
    </mode>
    
    <output>
    - Validation assessment with all issues identified
    - Updated spec documents (if issues resolved)
    - Summary of changes made
    </output>
  ",
  description="Validate spec documents"
)
```

## 4. Report Results

After validation completes, display summary:

```markdown
## Spec Validation Complete

**Documents Validated:**
- PRD.md ✓
- FRD.md ✓
- UserStories.md ✓

**Issues Found:** [N] issues
**Issues Resolved:** [N] issues

| Document | Issues Found | Issues Resolved |
|----------|--------------|-----------------|
| PRD.md | [N] | [N] |
| FRD.md | [N] | [N] |
| UserStories.md | [N] | [N] |

---
**Status:** Specs are ready for implementation ✓

**Next Steps:**
1. Review updated spec documents
2. Proceed with `/pivota_spec-plan-phase 1` when ready
```

**If issues remain unresolved:**
```markdown
**Status:** [N] issues require clarification

**Unresolved Issues:**
1. [Issue description] — [Document location]
2. [Issue description] — [Document location]

Run `/pivota_spec-validate-specs` again to resolve remaining issues.
```
</process>

<success_criteria>
- [ ] Spec documents exist before validation
- [ ] pivota_spec-validator agent spawned successfully
- [ ] All validation dimensions evaluated
- [ ] Issues identified with specific document locations
- [ ] User interviewed for clarification (unless --skip-interview)
- [ ] Original documents updated with resolved issues
- [ ] Validation summary provided
</success_criteria>

