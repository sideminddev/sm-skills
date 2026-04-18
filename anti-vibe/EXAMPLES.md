# Exemplos de Uso — Psters Workflow Skills

Guia prático de uso das skills em cenários comuns de desenvolvimento.

---

## Cenário 1: Nova Feature (Fluxo Completo)

**Contexto:** Você precisa implementar um "dashboard de métricas de vendas"

### Fase 1: Brainstorm

```
Skill: pwf-brainstorm
Input: "Dashboard de métricas de vendas em tempo real com gráficos e filtros"

Processo:
1. Agents executam: repo-research-analyst, integration-impact-analyst, 
   best-practices-researcher, learnings-researcher
2. Output: docs/brainstorms/20260118120000-sales-dashboard.md
3. Contém: opções de arquitetura, impacto em módulos, recomendação
```

### Fase 2: Planejamento

```
Skill: pwf-plan
Input: "Implementar dashboard de métricas de vendas"
Contexto: Ler brainstorm gerado na fase 1

Processo:
1. Carrega baseline de docs
2. Executa agents de research: repo-research-analyst, learnings-researcher, 
   spec-flow-analyzer
3. Executa agents de review: architecture-strategist, security-sentinel
4. Gera plano: docs/plans/20260118130000-sales-dashboard-plan.md
5. Contém: fases, tasks concretas, critérios de aceitação
```

### Fase 3: Execução — Fase 1

```
Skill: pwf-work-plan
Input: docs/plans/20260118130000-sales-dashboard-plan.md, Phase 1

Processo:
1. Lê o plano
2. Executa tasks da Fase 1 (ex: setup de módulo, DTOs, entidades)
3. Valida TypeScript: npm run validate
4. Atualiza documentação
5. Marca Fase 1 como completa no plano
```

### Fase 4: Execução — Fase 2

```
Skill: pwf-work-plan
Input: docs/plans/20260118130000-sales-dashboard-plan.md, Phase 2

Processo:
1. Continua com Fase 2 (ex: API endpoints, service layer)
2. Validação e documentação
```

### Fase 5: Review

```
Skill: pwf-review
Input: "Sales dashboard implementation — backend API and frontend components"

Processo:
1. Seleciona agents: nestjs-reviewer, angular-reviewer, security-sentinel, 
   performance-oracle
2. Executa em paralelo
3. Gera report com findings critical/important/informational
4. Você corrige findings críticos
```

### Fase 6: Commits

```
Skill: commit-changes
Input: scope das mudanças

Processo:
1. Agrupa mudanças por ticket/feature
2. Cria commits focados
3. Mensagens formatadas: "[TICKET-123] Add sales dashboard API endpoints"
```

---

## Cenário 2: Bug em Produção

**Contexto:** Erro 500 ao filtrar vendas por data

### Fase 1: Debug Sistemático

```
Skill: systematic-debugging
Input: "Erro 500 ao filtrar vendas por data no dashboard"

Processo:
1. Root-cause investigation:
   - Reproduzir erro com passos explícitos
   - Ler stack traces
   - Verificar mudanças recentes
   
2. Pattern identification:
   - Encontrar código similar funcionando
   - Comparar working vs failing
   
3. Hypothesis test:
   - Formar hipótese sobre causa
   - Aplicar menor mudança possível para testar
   
4. Minimal fix + validation:
   - Implementar fix da root-cause
   - Reproduzir e verificar fix
   - Confirmar sem regressões
```

### Fase 2: Validação

```
Skill: verification-before-completion

Evidência:
- Comando: npm run validate
- Result: exit 0
- Teste de reprodução: falha antes, passa depois
- Regressões: nenhuma detectada
```

### Fase 3: Documentação

```
Skill: pwf-doc-capture
Input: "Bug fix: date filtering error in sales dashboard"

Output: docs/solutions/20260118140000-date-filter-bug.md
Contém: problema, causa, solução, prevenção futura
```

---

## Cenário 3: Refactor de Componente

**Contexto:** Simplificar componente de usuário com 500+ linhas

### Fase 1: Análise

```
Skill: pwf-work
Input: "Refactor UserProfileComponent into smaller focused components"

Processo:
1. Load docs baseline
2. Executa repo-research-analyst para entender componente
3. Identifica: seções, responsabilidades, dependências
```

### Fase 2: Implementação

```
Tasks geradas:
- [ ] Criar UserHeaderComponent (avatar, nome, status)
- [ ] Criar UserDetailsComponent (info pessoal)
- [ ] Criar UserActionsComponent (botões de ação)
- [ ] Refatorar UserProfileComponent como container
- [ ] Atualizar testes
- [ ] Validar TypeScript
```

### Fase 3: Review

```
Skill: pwf-review
Input: "UserProfileComponent refactor"

Agents: angular-reviewer, code-simplicity-reviewer, julik-frontend-races-reviewer
```

---

## Cenário 4: Quick Fix (Trivial)

**Contexto:** Typo em mensagem de erro

```
Skill: pwf-work-light
Input: "Fix typo in error message at src/utils/validators.ts line 45"

Processo:
1. Lê arquivo afetado
2. Corrige typo
3. npm run validate
4. Commit (se necessário)

Tempo total: < 5 minutos
```

---

## Cenário 5: Setup de Novo Projeto

```
Skill: pwf-setup
Input: (nenhum — detecta automaticamente)

Processo:
1. Cria estrutura docs/:
   - docs/infrastructure/
   - docs/architecture/
   - docs/modules/
   - docs/features/
   - docs/solutions/patterns/
   - docs/plans/
   - docs/runbooks/
   - docs/workflow/

2. Cria arquivos baseline:
   - docs/infrastructure/README.md
   - docs/architecture/README.md
   - docs/workflow/operational-overrides.md (template)

3. Atualiza .gitignore para docs/
```

### Continuação: Foundation Docs

```
Skill: pwf-doc-foundation
Input: "all"

Processo:
1. Cria/atualiza:
   - docs/infrastructure.md
   - docs/architecture.md
   - docs/integrations.md
   - docs/environments.md
   - docs/glossary.md
```

---

## Cenário 6: TDD — Nova API Endpoint

**Contexto:** Implementar endpoint POST /api/reports/export

```
Skill: pwf-work-tdd
Input: "Create CSV export endpoint for sales reports"

Ciclo 1: Red
- Write test: should return CSV file for valid date range
- Test fails: endpoint doesn't exist

Ciclo 2: Green
- Implement minimal controller + service
- Test passes

Ciclo 3: Refactor
- Clean up code structure
- Add error handling
- Tests still pass

Repetir para:
- Input validation
- Error cases (invalid dates, no data)
- Authorization checks
- Performance (streaming large files)
```

---

## Cenário 7: Multi-Agente Paralelo

**Contexto:** Projeto grande afetando múltiplos módulos

```
Skill: orchestrating-multi-agents
Input: "Feature X affecting auth, billing, and notifications modules"

Processo:
1. Divide trabalho em sub-tasks independentes:
   - Sub-task A: Auth module changes
   - Sub-task B: Billing module changes
   - Sub-task C: Notifications module changes

2. Spawn agents em paralelo:
   - Cada agente executa pwf-work na sua sub-task
   - Resultados consolidados

3. Integration point:
   - Verificar contratos entre módulos
   - Testar integração
```

---

## Quick Reference: Quando Usar Cada Skill

| Situação | Skill |
|----------|-------|
| "Preciso planejar uma feature" | `pwf-plan` |
| "Tenho que fazer um pequeno ajuste" | `pwf-work-light` |
| "Bug em produção" | `systematic-debugging` |
| "Está na hora do code review" | `pwf-review` |
| "Preciso explorar opções primeiro" | `pwf-brainstorm` |
| "Vou implementar um plano existente" | `pwf-work-plan` |
| "Quero usar TDD" | `pwf-work-tdd` |
| "Preciso commitar as mudanças" | `commit-changes` |
| "Terminei, preciso verificar tudo" | `verification-before-completion` |
| "Preciso atualizar docs" | `pwf-doc` ou `docs-maintenance-after-work` |
| "Projeto novo, preciso de estrutura" | `pwf-setup` |
| "Preciso de worktree isolado" | `git-worktree` |
| "Branch terminado, o que fazer?" | `finishing-a-development-branch` |

---

## Dicas

1. **Sempre comece com `using-psters-workflow`** se não tiver certeza de qual skill usar

2. **Nunca pule documentação** — skills principais (`pwf-plan`, `pwf-work`, `pwf-work-plan`) já incluem manutenção de docs

3. **Use `pwf-work-light` apenas para trivial** — se durante execução descobrir que afeta >2 arquivos, pare e use `pwf-work`

4. **Validate antes de completion claims** — sempre use `verification-before-completion` ou equivalente

5. **Comente em tasks in_progress** — em Paperclip, comente antes de sair do heartbeat

---

**Para mais detalhes:** Veja `README.md` e documentação de cada skill em `skills/<name>/SKILL.md`
