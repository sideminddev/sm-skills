---
name: pwf-work
description: >
  USE WHEN: Executing free-form work outside formal plans — small fixes, minor
  adjustments, focused tasks. For planned phase execution, use pwf-work-plan instead.
  
  DON'T USE WHEN: The task is trivial/local-only (use pwf-work-light), tests-first
  is required (use pwf-work-tdd), or work belongs to an existing plan (use pwf-work-plan).
  
  REQUIRED INPUT: Description of what to do (e.g., "fix X", "improve Y", "add Z").
  
  OUTPUT: Implemented changes + updated documentation.
  
  PROCESS:
  1. Load docs baseline (mandatory — never skip)
  2. Classify scope (trivial vs non-trivial)
  3. Execute research agents (repo-research-analyst, learnings-researcher)
  4. Create task list, execute implementation
  5. Run TypeScript validation
  6. Update documentation (mandatory)
  7. Provide verification evidence
  
  NEXT STEPS: After completion, use pwf-review for full PR review, or pwf-commit-changes
  for structured commits.
---

# Execute Unplanned Work (Fast Path)

Use this skill for small fixes, minor adjustments, and focused tasks outside formal plans. For phase execution from `docs/plans/`, use `pwf-work-plan`.
Apply `using-psters-workflow` skill at start.

## Documentation Intent (Read This First)

Documentation in `docs/` is operational memory for future AI and engineers, not a release note.

Every documentation update must help a future implementation answer quickly:

- Where is the source of truth?
- What is implemented now vs only planned?
- Which invariants/rules cannot be broken?
- Which files/methods must change together?
- What are the known gotchas and safe change steps?

Avoid generic text that could apply to any project.

---

## Paperclip Integration

When running in Paperclip heartbeats:
- `PAPERCLIP_TASK_ID` contains the assigned issue (if triggered by task assignment)
- `PAPERCLIP_WAKE_REASON` indicates why the heartbeat started
- If `PAPERCLIP_WAKE_COMMENT_ID` is present, acknowledge the comment first
- Include `X-Paperclip-Run-Id: $PAPERCLIP_RUN_ID` on all API mutations

**Task Checkout (if assigned):**
Before starting work on an assigned task, checkout via Paperclip API:
```bash
curl -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
     -H "X-Paperclip-Run-Id: $PAPERCLIP_RUN_ID" \
     -X POST "$PAPERCLIP_API_URL/api/companies/$PAPERCLIP_COMPANY_ID/issues/$PAPERCLIP_TASK_ID/checkout"
```

---

## ⛔ MANDATORY WORKFLOW — NEVER SKIP ANY STEP

You MUST execute steps 1 through 6 IN ORDER. Do NOT jump to implementation.
Your FIRST action must be reading documentation (Step 1), NOT editing code.
If you skip Step 1 or Step 5, the workflow is BROKEN.

---

## Input

<work_description> #$ARGUMENTS </work_description>

If empty, ask: "What would you like me to work on?"

---

## Step 1: Research (BLOCKING — must complete before any code changes)

**Your first tool calls MUST be Read calls to load documentation. Do NOT start implementing.**

1. If the description is vague, ask one or two clarifying questions first.

0. **Classify scope first (trivial vs non-trivial):**
   - Trivial: <=2 files, no entities/migrations/endpoints/auth model changes.
   - If trivial, use lightweight path:
     - read only `docs/solutions/patterns/critical-patterns.md` (if exists) and directly relevant doc,
     - skip mandatory research-agent spawning,
     - keep focused implementation + verification evidence.
   - If non-trivial, follow full workflow below.

2. **Load docs baseline via skill (REQUIRED):**
   - Apply `docs-baseline-loading` skill.
   - Read mandatory baseline docs before implementation.
   - Create any missing canonical baseline docs immediately using the skill's required minimum sections.
   - Then read scope-specific docs:
     - `docs/solutions/patterns/critical-patterns.md` (if exists)
     - `docs/modules/<module>.md` (backend/module scope)
     - `docs/features/<feature>.md` (frontend/UI scope)
     - `docs/infrastructure/<component>.md` (infrastructure scope)
     - `docs/runbooks/README.md` (operational/deploy scope)
   - Search `docs/solutions/` by feature keywords for known gotchas.
   - **If any other expected doc file is missing:** create it immediately with a useful baseline (not placeholders). Minimum sections:
     - `Purpose` (business scope)
     - `Source of Truth Files` (exact paths)
     - `Current Implementation Snapshot` (what exists now)
     - `Planned/Upcoming Contract` (only if plan exists; clearly marked as planned)
     - `Invariants and Gotchas`
     - `Safe Change Checklist for Future AI Work`
     - `Related Plan and Docs`

3. **Check existing context:**
   - `docs/brainstorms/` — recent brainstorm for this feature?
   - `docs/plans/` — existing plan? If work belongs to a plan phase, suggest `pwf-work-plan` instead.

4. **Execute research agents (REQUIRED for non-trivial scope):**
   
   Execute the following agents by reading and applying their instructions:
   
   - **repo-research-analyst** (`../../agents/research/repo-research-analyst.md`)
     - Purpose: Maps file paths, existing patterns, rules, existing enums, migration state
     - Input: Feature description, affected modules
     - Output: Concrete file paths, existing patterns
   
   - **learnings-researcher** (`../../agents/research/learnings-researcher.md`)
     - Purpose: Surfaces relevant solutions from `docs/solutions/`
     - Input: Feature description, technical domain
     - Output: Applicable patterns, previous solutions
   
   Read both agent files and execute their instructions. You can read multiple files in parallel.

5. **Conditional research (execute applicable agents, non-trivial scope):**
   
   Execute the following agents when applicable by reading and applying their instructions:
   
   - Entity changes detected → **migration-impact-planner** (`../../agents/research/migration-impact-planner.md`)
   - Multi-step or UI flows → **spec-flow-analyzer** (`../../agents/workflow/spec-flow-analyzer.md`)
   - Security, payments, new tech → **best-practices-researcher** (`../../agents/research/best-practices-researcher.md`), **framework-docs-researcher** (`../../agents/research/framework-docs-researcher.md`)

6. **Present research summary to user:** Before implementing, show:
   - Files that will be changed (from research)
   - Relevant patterns/rules found
   - Any gotchas from `docs/solutions/`
   - Ask: "Do you have a ticket number (TICKET-XXXX) for commit messages?"

   Then proceed to Step 2.

---

## Step 2: Task List

1. Derive a task list — concrete, dependency-ordered.
   - Each task: **bold name + file path + sub-bullets** with method names, fields, classes.
   - No vague summaries. Every task must specify *what* to change in *which file*.

2. **Self-validate:** Review every task. Does it have a file path? Does it have specific method names or field names? If not, rewrite it.

3. **Debug route detection:** If this work is a bug/failure/regression fix:
   - First validate reproducibility with `bug-reproduction-validator` (`../../agents/workflow/bug-reproduction-validator.md`) when the report is ambiguous.
   - Then apply `systematic-debugging` skill (root-cause -> pattern -> hypothesis -> minimal fix) before implementing broad changes.
   - deep stack/source uncertainty -> `systematic-debugging/root-cause-tracing.md`
   - async timing/flaky behavior -> `systematic-debugging/condition-based-waiting.md`
   - regression hardening after fix -> `systematic-debugging/defense-in-depth.md`

### Built-in capabilities (use as needed during execution):

- **Operational policy source:** `../../references/rules/operational-guardrails.md`
- **Project overrides (optional):** `docs/workflow/operational-overrides.md` (if present, it overrides defaults from guardrails)
- **Database access:** Load DB vars from project `.env` file for database queries when applicable. Never display credentials.
- **Context7:** Use the Context7 MCP (`resolve-library-id` then `query-docs`) before implementing with external libraries.

---

## Step 3: Execute

For each task:

- Mark in progress.
- Read referenced files. Follow project rules and patterns from docs read in Step 1.
- Implement.
- Mark completed.

### ⚠️ CRITICAL: TypeORM Migration Atomic Chain (when applicable)

Follow the migration chain defined in `../../references/rules/operational-guardrails.md` (generate -> drift-check -> local run).
Treat this as blocking. Do not continue other tasks until the chain succeeds.

After all tasks:
- **TypeScript Validation** — Run `npm run validate` (or `tsc --noEmit` if no validate script). Fix ALL type/lint errors.
- **Build Only When Explicit** — Full build (`npm run build`) only runs when explicitly requested by user or during deployment workflows.

---

## Step 4: Quality Review

**Only run if 5+ files changed or multiple repos touched.** Otherwise skip to Step 5.

If triggered, execute review agents by reading and applying their instructions:

1. Read `../../references/review-agent-selection-mapping.md` to select applicable agents based on changed scope.
2. For each selected agent, read its agent file and execute the review instructions.
3. You can read multiple agent files in parallel.

Address **critical** findings only. Informational findings are noted but don't block.

---

## Step 5: Documentation Maintenance (MANDATORY — never skip)

**This step is MANDATORY even for small changes.**

Apply `docs-maintenance-after-work` skill and execute its full flow:
- always run `doc-shepherd`,
- run `plan-sync` when plan context exists,
- run specialized doc writers conditionally,
- run `pattern-extractor` when applicable,
- pass the documentation quality gate before Step 6.

---

## Step 6: Finish

Summarize: what was implemented, files changed, any caveats.

### Verification Evidence (MANDATORY before completion claims)

Before any "done/fixed/passing" claim, apply `verification-before-completion` skill and use the evidence format from `../../references/rules/operational-guardrails.md`.

Include a dedicated **Documentation updates** subsection listing:

- docs files updated/created
- what concrete knowledge was added (source-of-truth files, invariants, checklists, contracts)
- any remaining doc gaps explicitly marked for follow-up

Suggest:
- **Commit** with ticket number
- **`pwf-review`** for full PR review
- **`pwf-doc-capture`** if a non-trivial bug was fixed
- **`deploy-lambda`** reminder if Lambda repos were touched

## Conventions

- Follow canonical policy in `../../references/rules/operational-guardrails.md`.
- Follow commit policy in `../../references/rules/commits.md`.
- Use optional project overrides in `docs/workflow/operational-overrides.md` when present.

## Next Recommended Skills

- `pwf-work-light` for future trivial/local-only changes
- `pwf-review` for a full multi-agent review pass
- `pwf-commit-changes` after review approval
- `pwf-doc-capture` when a reusable fix/pattern emerged
- `finishing-a-development-branch` when branch/worktree is ready to close
