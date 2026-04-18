# Configuração do Adapter OpenCode

Este repositório de skills é otimizado para o **OpenCode adapter** do Paperclip com modelo **MiniMax**.

---

## Adapter: `opencode_local`

O OpenCode é o adapter padrão fornecido pelo Paperclip para execução local de agentes com integração nativa de skills.

### Configuração do Agente

No dashboard do Paperclip ou via API:

```json
{
  "name": "dev-agent",
  "adapterType": "opencode_local",
  "adapterConfig": {
    "model": "MiniMax",
    "skillsDir": "/path/to/sm-skills",
    "cwd": "/path/to/project",
    "timeoutSec": 300,
    "maxTurnsPerRun": 50
  }
}
```

### Variáveis de Ambiente Injetadas

O OpenCode adapter injeta automaticamente durante heartbeats:

```bash
# Identidade Paperclip
PAPERCLIP_AGENT_ID
PAPERCLIP_COMPANY_ID
PAPERCLIP_API_KEY
PAPERCLIP_API_URL
PAPERCLIP_RUN_ID

# Contexto de Wake
PAPERCLIP_WAKE_REASON
PAPERCLIP_TASK_ID
PAPERCLIP_WAKE_COMMENT_ID

# Configuração OpenCode
OPENCODE_MODEL=MiniMax
OPENCODE_SKILLS_DIR=/path/to/sm-skills
```

---

## Modelo: MiniMax

O modelo MiniMax é o LLM padrão usado pelo OpenCode adapter nesta configuração.

### Características

- **Contexto:** Grande janela de contexto para skills e documentação
- **Ferramentas:** Suporte a tool calling para agentes paralelos
- **Skills:** Injeção automática via `--add-dir`
- **Heartbeats:** Execução em janelas de tempo definidas

---

## Injeção de Skills

O OpenCode injecta skills automaticamente:

1. **Discovery:** Escaneia `skills/` no `skillsDir` configurado
2. **Routing:** Usa `name` e `description` do frontmatter para decidir quando aplicar
3. **Loading:** Carrega SKILL.md completo quando relevante para a task
4. **Execution:** Segue instruções na skill durante o heartbeat

### Estrutura Esperada

```
sm-skills/
└── skills/
    └── <skill-name>/
        ├── SKILL.md          # Documento principal (obrigatório)
        └── references/       # Arquivos auxiliares (opcional)
```

---

## Diferenças: OpenCode vs Outros Adapters

| Aspecto | OpenCode + MiniMax | Claude Local | Codex Local |
|---------|-------------------|--------------|-------------|
| **Provider** | Paperclip (nativo) | Anthropic | OpenAI |
| **Modelo** | MiniMax | Claude 3.x | GPT-4x |
| **Skills** | Injeção nativa | Via `--add-dir` | Via `--add-dir` |
| **Config** | Simplificada | Requer `ANTHROPIC_API_KEY` | Requer `OPENAI_API_KEY` |
| **Execução** | Otimizada para Paperclip | Genérica | Genérica |

---

## Exemplo de Uso

### Criar Agente via CLI

```bash
paperclipai agent create \
  --name "nestjs-dev" \
  --company-id "comp_123" \
  --adapter-type opencode_local \
  --adapter-config '{
    "model": "MiniMax",
    "skillsDir": "/home/user/sm-skills",
    "cwd": "/home/user/projects/my-app",
    "preset": "nestjs-api"
  }'
```

### Atualizar Agente Existente

```bash
paperclipai agent update <agent-id> \
  --adapter-type opencode_local \
  --adapter-config '{"model": "MiniMax", "skillsDir": "/path/to/sm-skills"}'
```

### Invocar Agente (Heartbeat Manual)

```bash
paperclipai run --agent <agent-id> --company-id <company-id> --watch
```

---

## Troubleshooting

### Skills não estão sendo carregadas

1. Verificar se `skillsDir` aponta para a raiz do repositório (onde está `skills/`)
2. Confirmar que cada skill tem `SKILL.md` com frontmatter YAML válido
3. Verificar logs do OpenCode: `paperclipai run --verbose`

### Modelo não responde como esperado

1. Verificar se `model` está configurado como `"MiniMax"`
2. Confirmar que o agente tem budget disponível
3. Verificar se o wake reason está correto (assignment, schedule, etc.)

### API calls falhando

1. Confirmar que `PAPERCLIP_API_KEY` está injetado
2. Verificar se `X-Paperclip-Run-Id` está incluído em mutações
3. Validar URL da API: `$PAPERCLIP_API_URL`

---

## Referências

- **Paperclip Adapters:** https://docs.paperclip.ing/adapters/overview
- **OpenCode Adapter:** https://docs.paperclip.ing/adapters/opencode-local
- **Writing Skills:** https://docs.paperclip.ing/guides/agent-developer/writing-a-skill
- **MiniMax Model:** [Documentação Paperclip interna]

---

**Configuração atual:** OpenCode adapter + MiniMax model  
**Última atualização:** 2026-04-18
