---
description: Generate UX mockup from UserStories. Creates wireframes, user flows, and interaction patterns.
argument-hint: "[--interactive]"
tools:
  read: true
  bash: true
  write: true
  task: true
  question: true
---
<context>
**Flags:**
- `--interactive` — Engage in UX dialogue before generating mockup (ask clarifying questions)
</context>

<objective>
Generate a comprehensive UX mockup document based on UserStories.

**Creates:**
- `project_specs/UX-Mockup.md` — Wireframes, user flows, interaction patterns

**Based on:**
- `project_specs/UserStories.md` — User stories with acceptance criteria
- `project_specs/PRD.md` — Product requirements
- `project_specs/FRD.md` — Functional requirements

**Output includes:**
- User flow diagrams
- Screen wireframes with layout descriptions
- Information hierarchy
- State designs (default, loading, success, error, empty)
- Interaction patterns
- Responsive considerations
- Accessibility notes
</objective>

<execution_context>
@/home/daytona/.config/opencode/agents/pivota_spec-ux-designer.md
</execution_context>

<process>
## 1. Prerequisites Check

**Verify required files exist:**
```bash
ls project_specs/UserStories.md 2>/dev/null
ls project_specs/PRD.md 2>/dev/null
```

**If UserStories.md doesn'\''t exist:**
```
Error: UserStories.md not found.

Generate user stories first:
/pivota_spec-generate-spec-docs

Or skip to UserStories generation and run this command after.
```

## 2. Spawn UX Designer Agent

Spawn the pivota_spec-ux-designer agent with context:

```
Task(
  subagent_type="pivota_spec-ux-designer",
  prompt="
    Generate UX-Mockup.md for this project.
    
    <context>
    Read these files:
    - project_specs/UserStories.md (required)
    - project_specs/PRD.md (required)
    - project_specs/FRD.md (if available)
    - .planning/PROJECT.md (project context)
    </context>
    
    <interactive_mode>
    [If --interactive flag is set]
    Before generating, ask the user clarifying questions about:
    - Information hierarchy preferences
    - User journey priorities
    - Interaction pattern preferences
    - Responsive priorities (mobile-first vs desktop-first)
    </interactive_mode>
    
    <output>
    Write to: project_specs/UX-Mockup.md
    
    Include:
    1. Overview of UX approach
    2. User flows for each major journey (ASCII diagrams)
    3. Screen designs with wireframe layouts
    4. Information hierarchy for each screen
    5. State designs (default, loading, success, error, empty)
    6. Interactive elements and behaviors
    7. Interaction patterns
    8. Responsive considerations
    9. Accessibility notes
    
    Reference user story IDs in each design section.
    </output>
    
    <quality_gate>
    - [ ] All user stories have corresponding UX designs
    - [ ] Flows are complete (entry to exit)
    - [ ] All states covered
    - [ ] Accessibility considered
    </quality_gate>
  ",
  description="Generate UX mockup from UserStories"
)
```

## 3. Verify Output

Check that `project_specs/UX-Mockup.md` was created and contains:
- User flows section
- Screen designs section
- States section
- Interaction patterns section

## 4. Report Completion

Display summary:

```markdown
## UX Mockup Generated ✓

**File:** project_specs/UX-Mockup.md

| Section | Status |
|---------|--------|
| User Flows | ✓ Generated |
| Screen Designs | ✓ Generated |
| State Designs | ✓ Generated |
| Interaction Patterns | ✓ Generated |
| Responsive Notes | ✓ Generated |
| Accessibility Notes | ✓ Generated |

**User Stories Covered:** [Count] stories

---
**Next Steps:**
1. Review UX-Mockup.md
2. Refine designs as needed
3. Use as reference during implementation
```
</process>

<success_criteria>
- [ ] UserStories.md exists before running
- [ ] pivota_spec-ux-designer agent spawned successfully
- [ ] UX-Mockup.md created with all required sections
- [ ] User stories are referenced in designs
- [ ] All states covered (default, loading, success, error, empty)
</success_criteria>

