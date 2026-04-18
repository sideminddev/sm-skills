---
name: pwf-work-plan
description: >
  USE WHEN: Executing a planned phase from docs/plans/. This skill runs one phase
  at a time with full documentation maintenance.
  
  DON'T USE WHEN: No plan exists (use pwf-plan first), or work is unplanned
  (use pwf-work), or task is trivial (use pwf-work-light).
  
  REQUIRED INPUT: Path to plan file and optionally phase number.
  
  OUTPUT: Completed phase with updated plan, documentation, and verification.
  
  PROCESS:
  1. Read the plan and identify the target phase
  2. Execute phase tasks in order
  3. Run TypeScript validation
  4. Update plan status and documentation
  5. Run review agents if needed
  6. Provide verification evidence
  
  NEXT STEPS: Continue with next phase using pwf-work-plan, or pwf-review
  before completing the plan.
---

# Execute Planned Phase

Use this skill to execute one phase from a plan in `docs/plans/`.

---

## Paperclip Integration

When running in Paperclip heartbeats with an assigned task:
- Use `PAPERCLIP_TASK_ID` to track which issue this work addresses
- Update task status via API as you progress through phase tasks
- Include `X-Paperclip-Run-Id` on all mutating operations

---

## Input

<plan_path> [phase_number] #$ARGUMENTS </plan_path>

If no phase specified, execute the first pending phase.

---

## Step 1: Load Plan

1. Read the plan file at `<plan_path>`
2. Identify target phase:
   - If phase specified: validate it exists and is pending
   - If not specified: find first phase with ⬜ Pending status
3. Verify phase dependencies are satisfied (previous phases marked ✅)
4. Read any referenced docs (brainstorm, clarifications, solutions)

---

## Step 2: Pre-Phase Setup

1. Apply `docs-baseline-loading` skill to ensure current docs context
2. Check for `docs/workflow/operational-overrides.md`
3. Load relevant `docs/modules/` or `docs/features/` for this phase scope
4. If entity/migration work detected, verify migration state

---

## Step 3: Execute Phase Tasks

For each task in the phase:

1. **Mark in-progress** in plan file (update task line with `⏳`)
2. **Read** all referenced files from the task
3. **Implement** following patterns from docs
4. **Mark completed** (update to `✅`)
5. **TypeScript check** after each significant file change

### Task Execution Rules:

- Follow exact file paths from task
- Implement specific changes described (method signatures, fields, DTOs)
- If task is unclear, read surrounding code for context
- Don't deviate from task scope without user approval

### Migration Atomic Chain (if applicable):

When a task involves entity changes:
1. Generate migration
2. Drift-check the generated migration
3. Run migration locally IMMEDIATELY
4. Verify before continuing other tasks

---

## Step 4: Phase Validation

After all phase tasks:

1. **TypeScript Validation**: `npm run validate` (or `tsc --noEmit`)
   - Fix ALL errors before claiming phase complete
2. **Build only if explicit**: Run `npm run build` only when requested

---

## Step 5: Quality Review (Conditional)

**Run if:** Phase touched 5+ files or spanned multiple repos

Execute review agents:

1. Determine which review agents apply based on changed scope:
   - NestJS changes → `nestjs-reviewer`
   - Next.js changes → `nextjs-reviewer`
   - Angular changes → `angular-reviewer`
   - Lambda changes → `lambda-reviewer`
   - DB changes → `data-integrity-guardian`, `schema-drift-detector`
   - Security areas → `security-sentinel`

2. Read and execute applicable agent files from `../../agents/review/`
3. Address critical findings
4. Informational findings: note but don't block

---

## Step 6: Documentation Update (MANDATORY)

Apply `docs-maintenance-after-work` skill:

1. Run `doc-shepherd` — update all affected docs
2. Run `plan-sync` — update this plan file:
   - Mark phase as `✅ Completed` in table
   - Update any "Current Implementation" sections
3. Run specialized doc writers if applicable:
   - Module docs for backend changes
   - Feature docs for frontend changes
   - Infrastructure docs for infra changes
4. Run `pattern-extractor` if reusable patterns emerged

---

## Step 7: Post-Phase

1. Update plan status table
2. Summarize: what was done, files changed, key decisions
3. Provide verification evidence:
   - Commands run and results
   - Files modified
   - Docs updated

### If more phases remain:

Present options:
- Continue to next phase with `pwf-work-plan <plan_path> [next_phase]`
- Run `pwf-review` before continuing (recommended for complex phases)
- Pause and resume later

### If all phases complete:

- Mark plan status as `completed`
- Suggest `pwf-review` for final validation
- Suggest `pwf-commit-changes` for structured commits
- Suggest `pwf-doc-capture` for any learnings/patterns

---

## Phase Completion Checklist

- [ ] All phase tasks executed
- [ ] TypeScript validation passes
- [ ] Plan file updated (phase marked complete)
- [ ] Documentation maintained
- [ ] Review completed (if triggered)
- [ ] Verification evidence provided

## Conventions

- Follow canonical policy in `../../references/rules/operational-guardrails.md`
- Follow commit policy in `../../references/rules/commits.md`
- Never mark phase complete without fresh validation

## Next Recommended Skills

- `pwf-work-plan <plan> [next_phase]` — continue execution
- `pwf-review` — full multi-agent review
- `pwf-commit-changes` — structured commits
- `finishing-a-development-branch` — when work complete
