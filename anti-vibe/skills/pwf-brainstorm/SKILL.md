---
name: pwf-brainstorm
description: >
  USE WHEN: Exploring a feature, scope, or architecture before committing to
  implementation. This skill runs research agents to surface options, constraints,
  and integration impact.
  
  DON'T USE WHEN: Ready to implement immediately (use pwf-plan directly), or
  doing trivial fixes (use pwf-work-light).
  
  REQUIRED INPUT: Feature idea, scope question, or architecture exploration topic.
  
  OUTPUT: Decision document saved to docs/brainstorms/ with architecture options,
  integration impact, and resolved questions.
  
  PROCESS:
  1. Execute research agents to explore feature space
  2. Identify integration impact across modules/repos
  3. Surface constraints and blockers
  4. Document architecture options with tradeoffs
  5. Capture resolved questions and decisions
  
  NEXT STEPS: Use pwf-plan to convert brainstorm into executable plan.
---

# Feature Exploration & Brainstorming

Use this skill to explore ideas, scope, and architecture options before committing to implementation.

Output: Decision document in `docs/brainstorms/` that feeds into `pwf-plan`.

---

## Paperclip Integration

This skill is often used in early planning stages without task assignment.
If exploring within Paperclip context, consider creating a tracking issue
for the brainstorm session to maintain company knowledge.

---

## Input

<exploration_topic> #$ARGUMENTS </exploration_topic>

Examples:
- "User dashboard with real-time metrics"
- "Should we use WebSockets or polling for notifications?"
- "Database schema for multi-tenant feature"

---

## Research Pack Execution

Execute the following agents in parallel by reading and applying their instructions:

### Core Exploration Agents

1. **repo-research-analyst** (`../../agents/research/repo-research-analyst.md`)
   - **Purpose:** Map existing architecture, find similar features, identify touchpoints
   - **Input:** Feature/scope description, suspected affected areas
   - **Output:** Existing patterns, relevant modules, potential integration points

2. **integration-impact-analyst** (`../../agents/research/integration-impact-analyst.md`)
   - **Purpose:** Identify cross-module and cross-repo dependencies
   - **Input:** Feature description, target modules
   - **Output:** Impact matrix: what changes where, contract implications

3. **best-practices-researcher** (`../../agents/research/best-practices-researcher.md`)
   - **Purpose:** Surface security, performance, and integration best practices
   - **Input:** Feature type, technology stack, integration requirements
   - **Output:** Patterns to follow, pitfalls to avoid

4. **learnings-researcher** (`../../agents/research/learnings-researcher.md`)
   - **Purpose:** Find previous solutions to similar problems
   - **Input:** Feature domain, technical keywords
   - **Output:** Applicable patterns from docs/solutions/

### Optional Deep-Dive Agents

5. **api-contract-designer** (`../../agents/research/api-contract-designer.md`)
   - **Condition:** When designing new API endpoints or data contracts
   - **Purpose:** Design REST/GraphQL contracts, DTO structures
   - **Output:** Proposed API spec with endpoints, request/response shapes

6. **data-model-designer** (`../../agents/research/data-model-designer.md`)
   - **Condition:** When schema/entity changes likely
   - **Purpose:** Design entity relationships, migrations
   - **Output:** Proposed schema with entities, relationships, migration strategy

7. **ux-consistency-reviewer** (`../../agents/research/ux-consistency-reviewer.md`)
   - **Condition:** For frontend/UI features
   - **Purpose:** Ensure UX patterns align with existing design system
   - **Output:** UX recommendations, component suggestions

8. **edge-case-hunter** (`../../agents/research/edge-case-hunter.md`)
   - **Purpose:** Find edge cases, error states, failure modes
   - **Output:** List of edge cases to handle in design

**Execution:** Read all applicable agent files and execute their instructions. You can read multiple files in parallel.

---

## Output Document

Write to `docs/brainstorms/<TIMESTAMP>-<topic-slug>.md` (`TIMESTAMP` = current time in `YYYYMMDDHHmmss`).

### Required Sections

1. **Topic / Question**
   - What are we exploring?
   - Why does it matter?

2. **Context & Constraints**
   - Current system state
   - Technical constraints
   - Business constraints

3. **Exploration Findings**
   - Summary from research agents
   - Key patterns discovered
   - Similar features in codebase

4. **Architecture Options**

   For each option:
   - **Option A: [Name]**
     - Description
     - Pros
     - Cons
     - Integration impact
     - Effort estimate (S/M/L)

5. **Integration Impact Matrix**

   | Component | Impact Level | Changes Required |
   |-----------|-------------|------------------|
   | Backend API | High/Med/Low | Description |
   | Database | High/Med/Low | Description |
   | Frontend | High/Med/Low | Description |
   | Lambdas | High/Med/Low | Description |

6. **Open Questions**
   - Questions that need answering before implementation
   - Who to ask
   - Blockers that require resolution

7. **Recommended Approach**
   - Which option is preferred and why
   - Prerequisites before implementation
   - Suggested next steps

8. **Related**
   - Links to research findings
   - References to similar solutions in docs/solutions/

---

## Post-Brainstorm Options

Present the brainstorm document and offer:

1. **Convert to plan**: Run `pwf-plan "[feature]"` using this brainstorm as context
2. **Clarify ambiguities**: Run `pwf-clarify [brainstorm-path]` on open questions
3. **Create checklist**: Run `pwf-checklist` for requirement quality gates
4. **Refine further**: Iterate on specific sections

---

## Conventions

- Brainstorms are decision support documents, not implementation specs
- Keep architecture options concrete (not hand-wavy)
- Document tradeoffs explicitly
- Link to evidence (file paths, patterns found)

## Next Recommended Skills

- `pwf-plan` — convert brainstorm to executable plan
- `pwf-clarify` — resolve open questions
- `pwf-checklist` — requirement quality gates
