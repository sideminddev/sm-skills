---
name: pwf-review
description: >
  USE WHEN: Running heavy multi-agent code review before PR or after significant
  implementation. This skill intentionally uses multiple review agents for thorough
  quality assurance.
  
  DON'T USE WHEN: Quick local check (use fast-validation), or during active
  development (review is end-of-work activity).
  
  REQUIRED INPUT: Scope of changes (files, features, or PR context).
  
  OUTPUT: Review report with critical, important, and informational findings.
  
  PROCESS:
  1. Determine applicable review agents based on changed scope
  2. Execute review agents in parallel
  3. Consolidate findings by severity
  4. Present actionable recommendations
  
  SCOPE: Intentionally heavy — use selectively, not for every change.
---

# Multi-Agent Code Review

Use this skill for thorough code review before opening a pull request.

**Intentionally heavy** — runs multiple specialized review agents. Use selectively.

---

## Paperclip Integration

Reviews can be triggered by task status changes or manual invocation.
If reviewing in Paperclip context, consider:
- Creating a review summary comment on the associated issue
- Linking findings to specific files/changes
- Escalating critical findings through chain of command

---

## Input

<review_scope> #$ARGUMENTS </review_scope>

Examples:
- Files changed: "src/auth/*, src/users/*"
- Feature scope: "Authentication refactor"
- PR context: "PR #123: User dashboard feature"

---

## Step 1: Select Review Agents

Based on changed scope, select applicable agents from `../../agents/review/`:

### Core Review Agents (Always Consider)

| Agent | File Path | When to Include |
|-------|-----------|-----------------|
| architecture-strategist | `../../agents/review/architecture-strategist.md` | Always — structural review |
| code-simplicity-reviewer | `../../agents/review/code-simplicity-reviewer.md` | Always — complexity check |

### Technology-Specific Agents

| Agent | When to Include |
|-------|-----------------|
| nestjs-reviewer | NestJS backend changes |
| nextjs-reviewer | Next.js full-stack changes |
| angular-reviewer | Angular frontend changes |
| lambda-reviewer | Lambda/pipeline changes |

### Domain-Specific Agents

| Agent | When to Include |
|-------|-----------------|
| security-sentinel | Auth, secrets, encryption, file upload |
| data-integrity-guardian | Database entities, migrations |
| schema-drift-detector | TypeORM migrations, schema changes |
| performance-oracle | DB queries, pagination, N+1 risks |
| deployment-verification-agent | Deployment-related changes |

### Specialized Reviewers

| Agent | When to Include |
|-------|-----------------|
| kieran-typescript-reviewer | Complex TypeScript patterns |
| julik-frontend-races-reviewer | RxJS, async frontend |
| agent-native-reviewer | Agent-native code patterns |
| pattern-recognition-specialist | Detect pattern violations |

---

## Step 2: Execute Review Agents

For each selected agent:

1. Read the agent file
2. Execute its instructions with:
   - Changed files/paths
   - Implementation context
   - Any specific concerns

**Parallel Execution:** Read and execute multiple agent files simultaneously.

---

## Step 3: Consolidate Findings

Organize findings by severity:

### Critical (Block Release)
- Security vulnerabilities
- Data integrity risks
- Broken contracts
- Type errors that escape to runtime

### Important (Address Before Merge)
- Architecture violations
- Performance issues
- Maintainability concerns
- Test coverage gaps

### Informational (Note, Don't Block)
- Style preferences
- Alternative approaches
- Future considerations
- Documentation suggestions

---

## Step 4: Present Review Report

Format:

```
## Review Report: [Scope]

### Summary
- Agents run: [list]
- Files reviewed: [count]
- Findings: X critical, Y important, Z informational

### Critical Findings (BLOCKING)
1. **[Finding Title]**
   - Location: `file.ts:line`
   - Issue: Description
   - Fix: Recommended approach

### Important Findings
1. **[Finding Title]**
   - Location: `file.ts:line`
   - Issue: Description
   - Recommendation: Suggested improvement

### Informational Findings
1. **[Finding Title]**
   - Note: Description
   - Optional: Suggestion

### Next Steps
- [ ] Address critical findings
- [ ] Consider important findings
- [ ] Re-run review after fixes
- [ ] Proceed to commit/pwf-commit-changes
```

---

## Step 5: Follow-Up

Based on findings:

**If critical findings:**
- Fix immediately
- Re-run affected review agents
- Verify fixes resolve issues

**If no critical findings:**
- Address important findings at discretion
- Proceed to `pwf-commit-changes`

---

## Conventions

- Always run architecture-strategist for structural review
- Include security-sentinel for any auth/security scope
- Don't skip domain-specific agents when applicable
- Distinguish critical from informational clearly

## Next Recommended Skills

- Fix critical findings → re-run `pwf-review`
- `pwf-commit-changes` — after review approval
- `pwf-doc-capture` — if patterns emerged during fixes
