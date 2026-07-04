---
description: Generate the canonical .pivota/start-dev.sh wrapper + .pivota/dev-script.meta.json for a project (Phase 37 framework workflow)
argument-hint: ""
tools:
  read: true
  write: true
  bash: true
  glob: true
  grep: true
---
<tool_rules>
**Bash tool:** Every call MUST include both `command` and `description` fields. Omitting `description` causes a schema validation error. Always provide a short description string.
</tool_rules>

<objective>
Inspect the current project repo, match its stack against the init-dev-server catalog, and write `.pivota/start-dev.sh` + `.pivota/dev-script.meta.json` per the framework spec. End with the platform-parsable sentinel `INIT-DEV-SERVER COMPLETE catalog_entry=<id> commit_sha=<sha>` so `dispatch_command` can confirm success.

**Hard constraints (do NOT improvise):**
- Wrapper path is ALWAYS `.pivota/start-dev.sh`. NEVER write `dev.sh` at repo root or any other location.
- Metadata path is ALWAYS `.pivota/dev-script.meta.json`.
- Both files MUST follow the schema in `meta-format.md` and `wrapper-template.md`.
- Catalog match is deterministic: single match → render that entry; zero match or multi-match → fall back to agent generation per D-06.
- Log destination in the wrapper is `/tmp/pivota-dev.log` (NEVER `/var/log/pivota/dev.log`).
- Wrapper shebang is `#!/usr/bin/env bash` with `set -euo pipefail`.
- Stage `.pivota/start-dev.sh` and `.pivota/dev-script.meta.json` together; commit with message `chore(pivota): generate dev-server startup script (catalog: <id>)`.
</objective>

<execution_context>
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/init-dev-server.md
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/init-dev-server/catalog/README.md
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/init-dev-server/meta-format.md
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/init-dev-server/wrapper-template.sh
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/init-dev-server/wrapper-template.md
@/home/daytona/.config/opencode/pivota_spec-framework/workflows/init-dev-server/multi-process-template.md
</execution_context>

<context>
Inputs available to the workflow are passed by `dispatch_command` in `backend/src/services/pivota_spec_service.rs:409` as the conversation context:
- `project_id` — the platform UUID of the project being initialized
- `workspace_id` — the workspace UUID
- `repo_path` — absolute filesystem path to the project'\''s working tree inside the sandbox
- `regen` — boolean; true when invoked from Workspace Settings → Regenerate

If no args were passed (regen is false / absent), discover the repo by reading the agent'\''s current working directory (`pwd`).
</context>

<process>
Execute the init-dev-server workflow from @/home/daytona/.config/opencode/pivota_spec-framework/workflows/init-dev-server.md end-to-end.

Follow its 4-step process verbatim:
1. **Inspect** — file-tree globs + manifest reads to gather `matched_globs`, `manifest_signatures`, `existing_scripts`.
2. **Match catalog** — single-match wins; multi-match falls through to agent fallback (D-06).
3. **Render or generate** — render the wrapper template with the matched catalog entry'\''s substitutions, OR (zero/multi-match) generate a project-specific wrapper following the wrapper-template.md contract.
4. **Write + commit** — write `.pivota/start-dev.sh` (mode 0755) and `.pivota/dev-script.meta.json`. `git add` BOTH paths, commit with `chore(pivota): generate dev-server startup script (catalog: <id>)`, then emit the success sentinel.

Preserve every locked decision (D-10, D-11.3, D-11.4, D-13, D-15, D-16, D-17) and avoid every anti-pattern listed in catalog/README.md.

Emit on the LAST line:
```
INIT-DEV-SERVER COMPLETE catalog_entry=<entry-id-or-agent-fallback> commit_sha=<short-sha>
```

The platform parses this sentinel — drift from this format makes the dispatch look like it never completed.
</process>

