# Psters Workflow Skills for PaperclipAI

Repositório de skills para agentes [PaperclipAI](https://paperclip.ing) via **OpenCode adapter** com modelo **MiniMax** — workflows estruturados de desenvolvimento com disciplina anti-vibe-coding.

---

## O que é este repositório?

Este repositório adapta o **Psters AI Workflow** (originalmente para Windsurf/Cursor) para o formato de skills do PaperclipAI, permitindo que agentes Paperclip utilizem workflows disciplinados de planejamento, execução, revisão e documentação.

**Origem:** Plugin de workflow diário de IA para desenvolvimento estruturado  
**Destino:** Skills para agentes PaperclipAI  
**Foco:** Anti-vibe-coding por design

---

## Instalação

### 1. Configurar no Paperclip (OpenCode + MiniMax)

No Paperclip, configure o agente com:

- **Adapter:** `opencode_local`
- **Model:** `MiniMax`
- **Skills Directory:** Path para este repositório

A configuração é feita via dashboard do Paperclip ou API:

```bash
# Exemplo de configuração de agente via CLI
paperclipai agent create \
  --name "dev-agent" \
  --adapter-type opencode_local \
  --adapter-config '{"model": "MiniMax", "skillsDir": "/path/to/sm-skills"}' \
  --company-id <company-id>
```

O adapter OpenCode injeta as skills automaticamente via `--add-dir` durante os heartbeats.

**📖 Veja o guia completo:** [`INTEGRATION_GUIDE.md`](INTEGRATION_GUIDE.md) — 3 métodos detalhados (Dashboard, CLI, API) com exemplos passo a passo.

### 2. Estrutura de Diretórios Esperada

```
sm-skills/
├── skills/              # Skills disponíveis
│   ├── pwf-plan/       # Planejamento
│   ├── pwf-work/     # Execução
│   └── ...
├── agents/             # Agentes especializados
│   ├── research/      # Agentes de pesquisa
│   ├── review/        # Agentes de revisão
│   └── ...
└── references/         # Documentação e regras
    ├── rules/         # Guardrails operacionais
    └── AGENTS.md      # Contrato de agentes
```

---

## Skills Disponíveis

### Workflow Principal

| Skill | Descrição | Quando Usar |
|-------|-----------|-------------|
| `pwf-plan` | Criar plano de implementação detalhado | Antes de começar feature complexa |
| `pwf-work` | Executar trabalho não planejado | Pequenos ajustes, fixes rápidos |
| `pwf-work-plan` | Executar fase de plano existente | Implementar plano criado por pwf-plan |
| `pwf-work-light` | Trabalho trivial (≤2 arquivos) | Quick fixes locais |
| `pwf-work-tdd` | Desenvolvimento test-first | Quando TDD é explicitamente solicitado |
| `pwf-review` | Review multi-agente | Antes de PR, após implementação grande |
| `pwf-brainstorm` | Explorar ideias/arquitetura | Fase inicial de descoberta |

### Documentação

| Skill | Descrição |
|-------|-----------|
| `pwf-doc` | Gerar/atualizar documentação técnica |
| `pwf-doc-foundation` | Criar baseline de docs do projeto |
| `pwf-doc-capture` | Capturar soluções e padrões |
| `docs-baseline-loading` | Carregar docs de contexto |
| `docs-maintenance-after-work` | Manter docs após implementação |

### Qualidade & Processo

| Skill | Descrição |
|-------|-----------|
| `systematic-debugging` | Debug estruturado (4 fases) |
| `verification-before-completion` | Verificação antes de claims |
| `fast-validation` | Validação TypeScript rápida |
| `commit-changes` | Commits estruturados |
| `git-worktree` | Gerenciar worktrees Git |
| `finishing-a-development-branch` | Finalizar branch/worktree |

### Frameworks & Tecnologias

| Skill | Descrição |
|-------|-----------|
| `nestjs-conventions` | Padrões NestJS |
| `nextjs-conventions` | Padrões Next.js |
| `angular-conventions` | Padrões Angular |
| `deploy-lambda` | Deploy de AWS Lambda |

---

## Como Usar

### Exemplo: Planejamento e Execução

```markdown
## Task: Implementar dashboard de métricas

### Fase 1: Planejamento
- Use skill: `pwf-plan`
- Input: "Implement user dashboard with real-time metrics"
- Output: `docs/plans/20260118120000-user-dashboard-plan.md`

### Fase 2: Execução
- Use skill: `pwf-work-plan`
- Input: path do plano, Phase 1
- Executar tasks, validar TypeScript
- Atualizar documentação

### Fase 3: Review
- Use skill: `pwf-review`
- Executar review agents
- Corrigir findings críticos

### Fase 4: Commit
- Use skill: `commit-changes`
- Criar commits focados com ticket numbers
```

### Exemplo: Debug

```markdown
## Task: Investigar erro em produção

- Use skill: `systematic-debugging`
- Seguir 4 fases: root-cause → pattern → hypothesis → fix
- Validar com `verification-before-completion`
- Documentar em `docs/solutions/`
```

---

## Princípios Anti-Vibe-Coding

1. **Contextualização primeiro**
   - Sempre ler docs antes de implementar
   - Nunca pular direto para código

2. **Documentação como memória operacional**
   - `docs/` é memória para futuros AI e engenheiros
   - Atualizar docs é parte obrigatória do workflow

3. **Estrutura e rastreabilidade**
   - Fases, tasks, review loops
   - Commits focados e ticket-aware

4. **Validação antes de completion claims**
   - Sem "done/fixed/passing" sem evidência de verificação
   - TypeScript validation é obrigatória

---

## Estrutura de uma Skill

Cada skill segue o formato Paperclip:

```markdown
---
name: skill-name
description: >
  USE WHEN: [situação de uso]
  DON'T USE WHEN: [quando não usar]
  REQUIRED INPUT: [o que é necessário]
  OUTPUT: [o que é produzido]
  PROCESS: [passos principais]
---

# Título da Skill

## Paperclip Integration
[Variáveis de ambiente e API calls]

## [Restante das instruções...]
```

---

## Integração Paperclip

### Variáveis de Ambiente

Quando executando em heartbeats Paperclip, estas variáveis estão disponíveis:

```bash
PAPERCLIP_AGENT_ID          # ID do agente
PAPERCLIP_COMPANY_ID        # Contexto da empresa
PAPERCLIP_API_KEY           # Token de autenticação
PAPERCLIP_API_URL           # URL da API
PAPERCLIP_RUN_ID            # ID do heartbeat atual
PAPERCLIP_TASK_ID           # Task atribuída (se houver)
PAPERCLIP_WAKE_REASON       # Por que o heartbeat iniciou
PAPERCLIP_WAKE_COMMENT_ID   # Comentário que disparou (se houver)
```

### API Calls

Exemplo de checkout de task:

```bash
curl -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
     -H "X-Paperclip-Run-Id: $PAPERCLIP_RUN_ID" \
     -X POST "$PAPERCLIP_API_URL/api/companies/$PAPERCLIP_COMPANY_ID/issues/$PAPERCLIP_TASK_ID/checkout"
```

---

## Adaptações do Plugin Original

| Aspecto | Plugin Windsurf | Paperclip Skills (OpenCode/MiniMax) |
|---------|-----------------|------------------|
| **Execução** | Direta no IDE | Paperclip heartbeats via OpenCode |
| **Adapter** | N/A | `opencode_local` |
| **Modelo** | N/A | `MiniMax` |
| **Comandos** | `/pwf-*` slash commands | Skills invocation |
| **Agentes** | Arquivos `.md` | Mesma estrutura, caminhos relativos |
| **Persistência** | Estado em arquivos | Paperclip API |
| **Hooks** | `afterFileEdit`, `stop` | Limitados pela arquitetura OpenCode |

### Funcionalidades Preservadas

✅ Todos os workflows (20 skills)  
✅ Todos os agentes especializados (45+)  
✅ Sistema de skills original  
✅ Princípios anti-vibe-coding  
✅ Documentação obrigatória  

### Funcionalidades Não Migradas

❌ Hooks automáticos de edição  
❌ Rastreamento `afterFileEdit`  
❌ Lembretes de shell `before/afterShellExecution`  

**Alternativa:** Lembretes manuais nas skills e disciplina da equipe.

---

## Referências

- **Documentação Paperclip:** https://docs.paperclip.ing
- **Writing a Skill:** https://docs.paperclip.ing/guides/agent-developer/writing-a-skill
- **How Agents Work:** https://docs.paperclip.ing/guides/agent-developer/how-agents-work
- **Core Concepts:** https://docs.paperclip.ing/start/core-concepts

---

## Contribuição

1. Skills devem seguir o formato Paperclip (YAML frontmatter + markdown)
2. Descrições devem funcionar como "routing logic" (USE WHEN / DON'T USE WHEN)
3. Incluir seção "Paperclip Integration" quando relevante
4. Manter princípios anti-vibe-coding

---

## Licença

[Adicionar licença apropriada]

---

**Versão:** 1.0  
**Última atualização:** 2026-04-18  
**Status:** Produção
