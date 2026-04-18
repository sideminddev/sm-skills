---
name: pwf-work-light
description: >
  USE WHEN: Executing trivial, local-only changes with minimal overhead.
  Ideal for quick fixes affecting <=2 files with no contract/schema/auth impact.
  
  DON'T USE WHEN: Changes affect >2 files, involve entities/migrations,
  API contracts, authentication models, or cross-repo work. Use pwf-work instead.
  
  DON'T USE WHEN: The task is part of a formal plan (use pwf-work-plan).
  
  REQUIRED INPUT: Specific change description with file paths.
  
  OUTPUT: Quick implementation with basic validation.
  
  PROCESS:
  1. Read only critical docs (patterns, relevant module/feature doc)
  2. Implement focused change
  3. Run fast validation (npm run validate)
  4. Minimal documentation update
  
  SCOPE LIMIT: ≤2 files, no contract changes, no migrations, local only.
---

# Execute Trivial Work (Light Path)

Use this skill for trivial/local-only changes with minimal overhead.

**Scope limit:** ≤2 files, no entities/migrations/endpoints/auth model changes.

---

## Paperclip Integration

This skill is optimized for quick heartbeats. Minimal API calls recommended.
If `PAPERCLIP_TASK_ID` is present, consider whether this trivial change should
be tracked as a task update or a quick fix.

---

## Input

<change_description> #$ARGUMENTS </change_description>

Example: "Fix typo in src/utils/helpers.ts line 45"

---

## Quick Workflow

### Step 1: Minimal Context (30 seconds max)

Read only:
- `docs/solutions/patterns/critical-patterns.md` (if exists)
- Directly relevant doc file (module/feature/infrastructure)

Skip full docs-baseline-loading.

### Step 2: Implement

- Make the specific change
- Keep to ≤2 files
- No migrations, no API contracts, no auth model changes

### Step 3: Validate

```bash
npm run validate
```

Fix type/lint errors only. Full build only if explicitly requested.

### Step 4: Document (Minimal)

Update only the directly affected doc file with:
- What changed
- Why (brief)
- Any gotchas for future changes

Skip full docs-maintenance-after-work cycle.

### Step 5: Finish

Quick summary:
- Files changed
- Validation result
- Any notes for follow-up

---

## When to STOP and Switch to pwf-work

If during execution you discover:
- Change affects >2 files
- Requires entity/migration work
- Touches API contracts or auth
- Has cross-repo impact

**STOP.** Tell user: "This is larger than trivial scope. Recommend switching to `pwf-work` for full workflow."

---

## Verification Evidence

Even for trivial changes, provide:
- `Command:` npm run validate
- `Result:` exit code 0 (or errors fixed)
- `Files:` list of modified files

---

## Conventions

- Follow `../../references/rules/operational-guardrails.md`
- Keep commits focused if committing
- Don't skip ALL documentation — just minimal update

## Next Recommended Skills

- `pwf-work` — if scope expands beyond trivial
- `pwf-review` — if >5 files touched (unlikely for light path)
- `pwf-commit-changes` — for structured commit
