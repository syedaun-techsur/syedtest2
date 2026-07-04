---
description: Stress-test technical architecture against 6 dimensions. Identifies critical decisions and validates robustness, simplicity, and codebase fit.
argument-hint: "[--skip-interview]"
tools:
  read: true
  bash: true
  write: true
  task: true
  question: true
---
<context>
**Flags:**
- `--skip-interview` — Report issues without asking clarifying questions
</context>

<objective>
Stress-test technical architecture decisions before implementation.

**Focus:** The critical 30% of decisions that shape 80-90% of implementation.

**Six Validation Dimensions:**
1. **Simplicity** — Is it as simple as it can be?
2. **Flexibility** — What if requirements change?
3. **Robustness** — What happens when components fail?
4. **Scaling** — What breaks under load?
5. **Codebase Fit** — Does it work with existing patterns?
6. **Consistency** — Does it address requirements?

**Process:**
1. Baseline coverage check
2. Identify 3-7 critical decisions
3. Stress-test each decision
4. Interview user to resolve issues
5. Update TechArch with clarifications
6. Confirm readiness for implementation

**Output:** Updated TechArch.md with stress-tested decisions.
</objective>

<execution_context>
@/home/daytona/.config/opencode/agents/pivota_spec-validator.md
</execution_context>

<process>
## 1. Prerequisites Check

**Verify TechArch exists:**
```bash
ls project_specs/TechArch.md 2>/dev/null
```

**If TechArch.md doesn'\''t exist:**
```
Error: TechArch.md not found.

Generate TechArch first:
/pivota_spec-generate-spec-docs
```

## 2. Spawn Validator Agent (TechArch Mode)

Spawn the pivota_spec-validator agent with TechArch mode:

```
Task(
  subagent_type="pivota_spec-validator",
  prompt="
    Validate TechArch with stress-testing mode.
    
    <mode>
    Mode: --techarch
    
    Use TechArch validation mode from <techarch_validation_mode> section.
    Focus on the critical 30% of architectural decisions.
    </mode>
    
    <context>
    Read these files:
    - project_specs/TechArch.md (required - architecture to validate)
    - project_specs/PRD.md (for requirements context)
    - project_specs/FRD.md (for requirements context)
    - project_specs/UserStories.md (for requirements context)
    - .planning/PROJECT.md (project context)
    
    Also analyze existing codebase patterns where applicable.
    </context>
    
    <validation_dimensions>
    Stress-test against these 6 dimensions:
    1. Simplicity — Is complexity justified?
    2. Flexibility — What if requirements change?
    3. Robustness — What breaks, what recovers?
    4. Scaling — Where are bottlenecks?
    5. Codebase Fit — Works with existing patterns?
    6. Consistency — Addresses all requirements?
    </validation_dimensions>
    
    <process>
    1. Baseline Coverage Check
       - Requirements coverage
       - Architecture completeness
       - Technical foundation
    
    2. Identify Critical Decisions (3-7 key choices)
       - Cross-component boundaries
       - Failure handling
       - Core data schemas
       - Pattern breaks
       - Performance implications
       - Security boundaries
    
    3. Stress-Test Each Decision
       - Trace requests end-to-end
       - Inject failures at key points
       - Change requirements — what ripples?
    
    4. Classify Issues
       - Most Important (blocks implementation)
       - Significant (needs resolution)
       - Moderate (should clarify)
       - Minor (observations)
    
    [If --skip-interview NOT set:]
    5. Interview user to resolve issues
    6. Update TechArch.md with clarifications
    7. Confirm readiness
    </process>
    
    <output>
    - Stress-test assessment of critical decisions
    - Issue classification by priority
    - Updated TechArch.md (if issues resolved)
    - Readiness confirmation
    </output>
  ",
  description="Validate TechArch with stress-testing"
)
```

## 3. Report Results

After validation completes, display summary:

```markdown
## TechArch Validation Complete

**Critical Decisions Analyzed:** [N] decisions
**Issues Found:** [N] issues
**Issues Resolved:** [N] issues

| Dimension | Issues Found | Status |
|-----------|--------------|--------|
| Simplicity | [N] | ✓ Clear / ⚠️ Needs Review |
| Flexibility | [N] | ✓ Clear / ⚠️ Needs Review |
| Robustness | [N] | ✓ Clear / ⚠️ Needs Review |
| Scaling | [N] | ✓ Clear / ⚠️ Needs Review |
| Codebase Fit | [N] | ✓ Clear / ⚠️ Needs Review |
| Consistency | [N] | ✓ Clear / ⚠️ Needs Review |

**Issues by Priority:**
- Most Important: [N]
- Significant: [N]
- Moderate: [N]
- Minor: [N]

---
**Status:** Architecture is ready for implementation ✓

**Next Steps:**
1. Review updated TechArch.md
2. Proceed with implementation planning
```

**If issues remain unresolved:**
```markdown
**Status:** [N] issues require clarification

**Unresolved Issues (by priority):**

**Most Important:**
1. [Issue description]

**Significant:**
1. [Issue description]

Run `/pivota_spec-validate-techarch` again to resolve remaining issues.
```
</process>

<success_criteria>
- [ ] TechArch.md exists before validation
- [ ] pivota_spec-validator spawned in TechArch mode
- [ ] Critical decisions identified (3-7 decisions)
- [ ] All 6 dimensions evaluated
- [ ] Issues classified by priority
- [ ] User interviewed for clarification (unless --skip-interview)
- [ ] TechArch.md updated with resolved issues
- [ ] Readiness status confirmed
</success_criteria>

