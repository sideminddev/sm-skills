# Agent Contract for PaperclipAI Integration

Este documento define o contrato operacional para agentes PaperclipAI utilizando as skills deste repositório.

---

## Core Principles

1. **User control first**
   - O usuário escolhe o caminho do workflow e os quality gates.
   - Não alternar silenciosamente entre skills.

2. **Predictable and repeatable execution**
   - Preferir passos explícitos e outputs determinísticos.
   - Manter comandos com side-effect explícitos e não-automáticos.

3. **Durable documentation**
   - `docs/` é memória operacional.
   - Manter docs alinhados com estado implementado.

---

## Paperclip Integration

### Environment Variables

Em heartbeats Paperclip, estas variáveis são injetadas automaticamente:

```bash
# Identidade
PAPERCLIP_AGENT_ID          # Seu ID de agente
PAPERCLIP_COMPANY_ID        # ID da empresa

# API
PAPERCLIP_API_URL           # URL base da API (ex: http://localhost:3100/api)
PAPERCLIP_API_KEY           # Token JWT para autenticação

# Contexto do Run
PAPERCLIP_RUN_ID            # ID do heartbeat atual
PAPERCLIP_WAKE_REASON       # Por que despertou: schedule, assignment, comment, manual, approval

# Contexto de Task (quando aplicável)
PAPERCLIP_TASK_ID           # Task atribuída que disparou o heartbeat
PAPERCLIP_WAKE_COMMENT_ID   # Comentário específico que disparou
PAPERCLIP_APPROVAL_ID       # ID de aprovação (se wake foi por approval)
PAPERCLIP_APPROVAL_STATUS   # approved ou rejected
PAPERCLIP_WAKE_PAYLOAD_JSON # Payload compacto de issue e comentários (usar primeiro)

# Linked Issues
PAPERCLIP_LINKED_ISSUE_IDS  # IDs de issues vinculadas (comma-separated)
```

### Heartbeat Protocol

1. **Wake** → algo dispara o agente (schedule, assignment, @mention, invoke manual)
2. **Adapter invocation** → Paperclip chama o adapter configurado
3. **Agent process** → adapter inicia o runtime do agente
4. **Paperclip API calls** → agente verifica assignments, claima tasks, faz trabalho
5. **Result capture** → adapter captura output, uso, custos, estado de sessão
6. **Run record** → Paperclip armazena o resultado para audit/debugging

### Task Checkout (Obrigatório)

**Sempre fazer checkout antes de trabalhar.** Nunca PATCH manualmente para `in_progress`.

```bash
curl -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
     -H "X-Paperclip-Run-Id: $PAPERCLIP_RUN_ID" \
     -X POST "$PAPERCLIP_API_URL/api/companies/$PAPERCLIP_COMPANY_ID/issues/$PAPERCLIP_TASK_ID/checkout"
```

### Status Lifecycle

```
backlog → todo → in_progress → in_review → done | blocked
```

**Regras críticas:**
- **Sempre fazer checkout** antes de trabalhar
- **Nunca retry um 409** → a task pertence a outro agente
- **Nunca procurar trabalho não-assigned**
- **Auto-assign apenas para handoff explícito** (requer @-mention)
- **Sempre comentar** em work `in_progress` antes de sair do heartbeat
- **Sempre setar `parentId`** em subtasks

---

## Agent Organization

Agentes são organizados por domínio em `agents/`:

### Research Agents (`agents/research/`)

Exploração e gathering de informação:

| Agente | Propósito |
|--------|-----------|
| `repo-research-analyst` | Mapear file paths, serviços, DTOs, padrões |
| `learnings-researcher` | Buscar solutions em `docs/solutions/` |
| `migration-impact-planner` | Planejar migrations TypeORM/Prisma |
| `spec-flow-analyzer` | Análise de flows, edge cases, critérios de aceitação |
| `best-practices-researcher` | Padrões de segurança e integração |
| `framework-docs-researcher` | Padrões específicos de frameworks |
| `git-history-analyzer` | Contexto histórico para refactors |
| `integration-impact-analyst` | Impacto cross-module/repo |
| `api-contract-designer` | Design de contratos API |
| `data-model-designer` | Design de schema/entidades |
| `edge-case-hunter` | Encontrar edge cases |
| `ux-consistency-reviewer` | Consistência de UX |
| `po-analyst` | Análise de requisitos |
| `lambda-pipeline-analyst` | Análise de pipelines Lambda |

### Review Agents (`agents/review/`)

Quality assurance e revisão:

| Agente | Propósito |
|--------|-----------|
| `architecture-strategist` | Review estrutural, boundaries |
| `code-simplicity-reviewer` | Complexidade e simplicidade |
| `security-sentinel` | Riscos de segurança |
| `performance-oracle` | Otimização e DB queries |
| `nestjs-reviewer` | Padrões NestJS |
| `nextjs-reviewer` | Padrões Next.js |
| `angular-reviewer` | Padrões Angular |
| `lambda-reviewer` | Padrões Lambda |
| `data-integrity-guardian` | Integridade de dados |
| `schema-drift-detector` | Detecção de schema drift |
| `kieran-typescript-reviewer` | Padrões TypeScript |
| `julik-frontend-races-reviewer` | RxJS, async frontend |
| `agent-native-reviewer` | Padrões agent-native |
| `deployment-verification-agent` | Verificação de deploy |
| `pattern-recognition-specialist` | Detecção de violações de padrão |

### Workflow Agents (`agents/workflow/`)

Orquestração de workflow:

| Agente | Propósito |
|--------|-----------|
| `spec-flow-analyzer` | Análise de especificações |
| `plan-document-reviewer` | Review de documentos de plano |
| `bug-reproduction-validator` | Validar reprodução de bugs |

---

## Skills Integration

### Meta-Skill: `using-psters-workflow`

Use esta skill para decidir qual workflow skill aplicar antes de responder ou implementar.

**Regra:** Se há 1% de chance de uma skill aplicar, use-a.

**Priority order:**
1. Process discipline: `systematic-debugging`, `verification-before-completion`
2. Execution-context: `git-worktree`, `finishing-a-development-branch`, `orchestrating-multi-agents`
3. Domain/convention: `nestjs-conventions`, `angular-conventions`, `deploy-lambda`, `commit-changes`

### Quick Routing

- Bug ou comportamento falhando → `systematic-debugging`
- Fim de implementação → `verification-before-completion`
- Branch isolado paralelo → `git-worktree`
- Multi-subagent paralelização → `orchestrating-multi-agents`
- Branch finalizado, decisão necessária → `finishing-a-development-branch`

---

## Side-Effect Command Policy

Comandos que escrevem arquivos, fazem deploy, ou commit devem ser ações explícitas do usuário:

**Side-effect skills:**
- `pwf-work`, `pwf-work-plan`, `pwf-work-light`, `pwf-work-tdd`
- `pwf-plan`, `pwf-brainstorm`, `pwf-clarify`, `pwf-checklist`, `pwf-analyze`
- `pwf-doc`, `pwf-doc-foundation`, `pwf-doc-runbook`
- `pwf-doc-capture`, `pwf-doc-refresh`
- `pwf-setup`, `pwf-setup-workspace`, `pwf-commit-changes`, `deploy-lambda`

---

## Multi-Agent Orchestration

Subagentes paralelos são encorajados quando tasks são independentes.

**Guardrails:**
- Manter um orchestrator owner
- Usar explicit role boundaries
- Merge outputs em uma decisão determinística
- Perguntar ao usuário antes de execução autônoma de alto risco

---

## Minimal Validation Before Release/Commit

Sempre rodar:
1. Lint/diagnostic checks para arquivos editados
2. Fast validation: `npm run validate` (ou equivalente do projeto)
3. Docs consistency check para workflow commands atualizados

---

## Commit Co-author

Se você fizer um git commit, você DEVE adicionar EXATAMENTE:

```
Co-Authored-By: Paperclip <noreply@paperclip.ing>
```

Ao final de cada mensagem de commit. Não colocar seu nome de agente.

---

## Critical Rules (from Paperclip)

- **Always checkout** before working
- **Never retry a 409** → task belongs to someone else
- **Never look for unassigned work**
- **Self-assign only for explicit @-mention handoff**
- **Honor "send it back to me" requests** from board users
- **Always comment** on `in_progress` work before exiting
- **Always set `parentId`** on subtasks
- **Preserve workspace continuity** for follow-ups
- **Never cancel cross-team tasks** → reassign to manager
- **Always update blocked issues explicitly**
- **Use first-class blockers** via `blockedByIssueIds`
- **@mentions** trigger heartbeats — use sparingly (cost budget)
- **Budget**: auto-paused at 100%, focus on critical above 80%
- **Escalate** via `chainOfCommand` when stuck

---

## Governance

- **Hiring agents**: agents podem solicitar contratação de subordinados, mas board deve aprovar
- **CEO strategy**: plano estratégico inicial do CEO requer board approval
- **Board overrides**: board pode pausar, resumir, ou terminar qualquer agente e reassign qualquer task

---

## Path Conventions

Usar logical plugin paths para assets internos:
- `skills/...`, `agents/...`, `references/rules/...`

Usar project-owned paths para dados do repositório do usuário:
- `docs/...` (na raiz do repositório alvo)

---

## Differences from Original Plugin

| Aspecto | Plugin Windsurf | Paperclip Skills |
|---------|-----------------|------------------|
| **Execução** | Direta no IDE | Heartbeats via adapter |
| **Contexto** | Sessão contínua | Janelas curtas de execução |
| **Variáveis** | Ambiente IDE | `PAPERCLIP_*` env vars |
| **Persistência** | Estado em arquivos | Paperclip API |
| **Hooks** | `afterFileEdit`, `stop` | Não disponível |

---

## References

- **Paperclip Core Concepts:** https://docs.paperclip.ing/start/core-concepts
- **How Agents Work:** https://docs.paperclip.ing/guides/agent-developer/how-agents-work
- **Heartbeat Protocol:** https://docs.paperclip.ing/guides/agent-developer/heartbeat-protocol
- **Writing a Skill:** https://docs.paperclip.ing/guides/agent-developer/writing-a-skill
- **Task Workflow:** https://docs.paperclip.ing/guides/agent-developer/task-workflow

---

**Versão:** 1.0  
**Última atualização:** 2026-04-18
