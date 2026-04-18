# Changelog

Todas as mudanças notáveis neste repositório de skills serão documentadas neste arquivo.

## [1.0.0] - 2026-04-18

### Added
- **Skills de Workflow Principal (8)**
  - `pwf-plan` — Criação de planos de implementação detalhados
  - `pwf-work` — Execução de trabalho não-planejado
  - `pwf-work-plan` — Execução de fases de planos existentes
  - `pwf-work-light` — Trabalho trivial com overhead mínimo
  - `pwf-work-tdd` — Desenvolvimento test-first
  - `pwf-review` — Review multi-agente
  - `pwf-brainstorm` — Exploração de features e arquitetura
  - `pwf-setup` — Setup inicial de projeto

- **Skills de Documentação (5)**
  - `pwf-doc` — Geração de documentação técnica
  - `pwf-doc-foundation` — Baseline de documentação
  - `docs-baseline-loading` — Carregamento de contexto
  - `docs-maintenance-after-work` — Manutenção pós-implementação

- **Skills de Qualidade e Processo (8)**
  - `systematic-debugging` — Debug estruturado em 4 fases
  - `verification-before-completion` — Verificação obrigatória
  - `fast-validation` — Validação TypeScript rápida
  - `commit-changes` — Commits estruturados
  - `git-worktree` — Gerenciamento de worktrees
  - `finishing-a-development-branch` — Finalização de branches
  - `orchestrating-multi-agents` — Orquestração paralela

- **Skills de Framework (4)**
  - `nestjs-conventions` — Padrões NestJS
  - `nextjs-conventions` — Padrões Next.js
  - `angular-conventions` — Padrões Angular
  - `deploy-lambda` — Deploy AWS Lambda

- **Meta-Skill (1)**
  - `using-psters-workflow` — Seleção de workflow adequado

- **Agentes (45+)**
  - 14 agentes de research
  - 15 agentes de review
  - 3+ agentes de workflow

- **Documentação**
  - `README.md` — Documentação principal em português/inglês
  - `references/AGENTS.md` — Contrato operacional para PaperclipAI
  - `.paperclip/skill-manifest.json` — Manifest de skills
  - `ADAPTER_CONFIG.md` — Guia de configuração OpenCode + MiniMax
  - Regras operacionais em `references/rules/`

### Adaptações para Paperclip

- Conversão de comandos `/pwf-*` para skills
- Adaptação de frontmatter para formato Paperclip
- Adição de seção "Paperclip Integration" em skills principais
- Mapeamento de variáveis `PAPERCLIP_*` env
- Documentação de heartbeat protocol
- Referências a agentes com caminhos relativos (`../../agents/...`)

### Configuração de Adapter e Modelo

- **Adapter:** `opencode_local` (OpenCode adapter nativo do Paperclip)
- **Modelo:** `MiniMax` (LLM otimizado para Paperclip heartbeats)
- **Injeção de Skills:** Automática via `--add-dir` durante heartbeats
- **Suporte:** Skills discovery, routing por description, loading dinâmico

### Preservado do Plugin Original

- 20 workflows → 20+ skills
- 45+ agentes especializados
- Princípios anti-vibe-coding
- Documentação obrigatória
- Validação TypeScript
- Sistema de presets

### Não Migrado

- Hooks automáticos (`afterFileEdit`, `stop`)
- Rastreamento automático de edições
- Lembretes de shell (`before/afterShellExecution`)

---

## Modelo de Versionamento

Este projeto segue [Semantic Versioning](https://semver.org/):

- `MAJOR` — Mudanças incompatíveis na estrutura de skills
- `MINOR` — Adição de novas skills ou features
- `PATCH` — Correções e melhorias em skills existentes

---

## [Próximas Versões]

### [1.1.0] - Planejado

- [ ] Skill `pwf-analyze` — Análise read-only de consistência
- [ ] Skill `pwf-clarify` — Remoção de ambiguidades
- [ ] Skill `pwf-checklist` — Quality gates por domínio
- [ ] Suporte a hooks Paperclip (quando disponível)

### [1.2.0] - Planejado

- [ ] Integração com Paperclip API para status updates
- [ ] Skill `visual-brainstorm-companion` — Brainstorm visual
- [ ] Templates de plano por preset
- [ ] Documentação em vídeo/guia interativo

---

## Contribuições

Para contribuir com novas skills ou melhorias:

1. Siga o formato Paperclip (YAML frontmatter + markdown)
2. Inclua "USE WHEN / DON'T USE WHEN" na description
3. Adicione seção "Paperclip Integration" quando relevante
4. Mantenha princípios anti-vibe-coding
5. Atualize este CHANGELOG

---

**Versão atual:** 1.0.0  
**Última atualização:** 2026-04-18
