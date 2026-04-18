# Guia de Integração: Psters Workflow + Paperclip

Este guia apresenta **3 formas** de integrar as skills do Psters Workflow ao Paperclip usando o adapter **OpenCode** com modelo **MiniMax**.

---

## Método 1: Dashboard do Paperclip (UI Web) — Mais Simples

### Passo 1: Acessar o Dashboard

1. Acesse: `https://app.paperclip.ing` (ou sua instância local)
2. Faça login com suas credenciais
3. Selecione sua **Company**

### Passo 2: Criar Novo Agente

1. Navegue até **Agents** → **Hire Agent** (ou **+ New Agent**)
2. Preencha os dados básicos:
   - **Name:** `dev-agent` (ou nome descritivo)
   - **Role:** `engineer` (ou role apropriado)
   - **Title:** `Software Engineer`
   - **Reports To:** Selecione o manager (ou deixe vazio para CEO)
   - **Capabilities:** Descrição das capacidades

### Passo 3: Configurar Adapter

Na seção **Adapter Configuration**:

1. **Adapter Type:** Selecione `opencode_local`
2. **Model:** `MiniMax`
3. **Working Directory (cwd):** `/path/to/your/project`

### Passo 4: Configurar Skills

Na seção **Skills** (ou **Desired Skills**):

1. **Skills Directory:** `/path/to/sm-skills`
2. O Paperclip irá automaticamente descobrir todas as skills em `skills/`

Ou especifique skills individuais:
```
pwf-plan
pwf-work
systematic-debugging
```

### Passo 5: Configurar Runtime

Na seção **Runtime Configuration**:

```json
{
  "heartbeat": {
    "enabled": true,
    "intervalSec": 300,
    "wakeOnDemand": true,
    "wakeOnMention": true
  }
}
```

### Passo 6: Finalizar

1. **Budget Monthly (cents):** Defina o orçamento (ex: `10000` = $100)
2. Clique em **Hire Agent** ou **Create**

### Passo 7: Testar

1. O agente aparecerá no dashboard
2. Atribua uma task de teste ou clique **Wake Now**
3. Verifique se o agente carrega as skills corretamente

---

## Método 2: CLI (Command Line) — Rápido & Automatizado

### Pré-requisitos

```bash
# Instalar Paperclip CLI
npm install -g @paperclipai/cli

# Ou via npx
npx @paperclipai/cli --version
```

### Passo 1: Configurar Variáveis de Ambiente

```bash
export PAPERCLIP_API_KEY="seu-api-key"
export PAPERCLIP_API_URL="http://localhost:3100/api"  # ou produção
export PAPERCLIP_COMPANY_ID="comp_seu-id"
```

### Passo 2: Criar Agente

#### Opção A: Comando Simples

```bash
paperclipai agent create \
  --name "dev-agent" \
  --role "engineer" \
  --title "Software Engineer" \
  --adapter-type opencode_local \
  --adapter-config '{
    "model": "MiniMax",
    "cwd": "/path/to/your/project"
  }' \
  --budget-monthly-cents 10000 \
  --company-id $PAPERCLIP_COMPANY_ID
```

#### Opção B: Comando com Skills Específicas

```bash
paperclipai agent hire \
  --name "nestjs-dev" \
  --role "backend-engineer" \
  --adapter-type opencode_local \
  --adapter-config '{
    "model": "MiniMax",
    "cwd": "/path/to/nestjs-project",
    "skillsDir": "/path/to/sm-skills"
  }' \
  --desired-skills 'pwf-plan,pwf-work,nestjs-conventions,systematic-debugging' \
  --company-id $PAPERCLIP_COMPANY_ID
```

#### Opção C: Usando arquivo JSON

Crie `agent-config.json`:

```json
{
  "name": "fullstack-dev",
  "role": "senior-engineer",
  "title": "Senior Fullstack Engineer",
  "reportsTo": null,
  "capabilities": "Full-stack development with NestJS and Angular",
  "adapterType": "opencode_local",
  "adapterConfig": {
    "model": "MiniMax",
    "cwd": "/home/user/projects/main-app",
    "skillsDir": "/home/user/sm-skills"
  },
  "desiredSkills": [
    "pwf-plan",
    "pwf-work",
    "pwf-work-plan",
    "nestjs-conventions",
    "angular-conventions",
    "systematic-debugging",
    "commit-changes"
  ],
  "runtimeConfig": {
    "heartbeat": {
      "enabled": true,
      "intervalSec": 300,
      "wakeOnDemand": true
    }
  },
  "budgetMonthlyCents": 15000
}
```

Execute:
```bash
paperclipai agent create --from-file agent-config.json --company-id $PAPERCLIP_COMPANY_ID
```

### Passo 3: Verificar Criação

```bash
# Listar agentes da company
paperclipai agent list --company-id $PAPERCLIP_COMPANY_ID

# Ver detalhes do agente criado
paperclipai agent get <agent-id> --company-id $PAPERCLIP_COMPANY_ID
```

### Passo 4: Testar Agente

```bash
# Acordar agente manualmente
paperclipai agent wake <agent-id> --company-id $PAPERCLIP_COMPANY_ID

# Ou com watch para ver execução em tempo real
paperclipai run --agent <agent-id> --company-id $PAPERCLIP_COMPANY_ID --watch
```

---

## Método 3: API REST — Controle Total Programático

### Pré-requisitos

```bash
export PAPERCLIP_API_KEY="seu-api-key"
export PAPERCLIP_API_URL="http://localhost:3100/api"
export PAPERCLIP_COMPANY_ID="comp_seu-id"
```

### Endpoint: Criar Agente

```bash
curl -X POST "$PAPERCLIP_API_URL/api/companies/$PAPERCLIP_COMPANY_ID/agent-hires" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "dev-agent",
    "role": "engineer",
    "title": "Software Engineer",
    "reportsTo": null,
    "capabilities": "Full-stack development with structured workflows",
    "adapterType": "opencode_local",
    "adapterConfig": {
      "model": "MiniMax",
      "cwd": "/path/to/project",
      "skillsDir": "/path/to/sm-skills"
    },
    "desiredSkills": [
      "pwf-plan",
      "pwf-work",
      "pwf-work-plan",
      "systematic-debugging",
      "nestjs-conventions",
      "commit-changes"
    ],
    "runtimeConfig": {
      "heartbeat": {
        "enabled": true,
        "intervalSec": 300,
        "wakeOnDemand": true,
        "wakeOnMention": true
      }
    },
    "budgetMonthlyCents": 10000
  }'
```

### Resposta Esperada

```json
{
  "message": "Agent hire request submitted.",
  "agentId": "agent-abc123",
  "status": "pending"
}
```

### Script de Automação Completo

Crie `setup-agent.sh`:

```bash
#!/bin/bash
set -e

# Configurações
PAPERCLIP_API_KEY="${PAPERCLIP_API_KEY:-$1}"
PAPERCLIP_COMPANY_ID="${PAPERCLIP_COMPANY_ID:-$2}"
PAPERCLIP_API_URL="${PAPERCLIP_API_URL:-http://localhost:3100/api}"

SKILLS_DIR="${SKILLS_DIR:-/path/to/sm-skills}"
PROJECT_DIR="${PROJECT_DIR:-/path/to/project}"

AGENT_NAME="${AGENT_NAME:-psters-dev}"
AGENT_ROLE="${AGENT_ROLE:-engineer}"

# Criar payload
cat > /tmp/agent-payload.json <<EOF
{
  "name": "$AGENT_NAME",
  "role": "$AGENT_ROLE",
  "title": "Software Engineer",
  "capabilities": "Development with Psters Workflow skills",
  "adapterType": "opencode_local",
  "adapterConfig": {
    "model": "MiniMax",
    "cwd": "$PROJECT_DIR",
    "skillsDir": "$SKILLS_DIR"
  },
  "desiredSkills": [
    "pwf-plan",
    "pwf-work",
    "pwf-work-plan",
    "pwf-brainstorm",
    "pwf-review",
    "systematic-debugging",
    "commit-changes",
    "nestjs-conventions",
    "fast-validation"
  ],
  "runtimeConfig": {
    "heartbeat": {
      "enabled": true,
      "intervalSec": 300,
      "wakeOnDemand": true
    }
  },
  "budgetMonthlyCents": 10000
}
EOF

echo "🚀 Criando agente $AGENT_NAME..."

# Chamada API
RESPONSE=$(curl -s -X POST \
  "$PAPERCLIP_API_URL/api/companies/$PAPERCLIP_COMPANY_ID/agent-hires" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "Content-Type: application/json" \
  -d @/tmp/agent-payload.json)

# Extrair agentId
AGENT_ID=$(echo $RESPONSE | grep -o '"agentId":"[^"]*"' | cut -d'"' -f4)

if [ -n "$AGENT_ID" ]; then
  echo "✅ Agente criado: $AGENT_ID"
  echo ""
  echo "Próximos passos:"
  echo "  1. Atribuir uma task ao agente no dashboard"
  echo "  2. Acordar: paperclipai agent wake $AGENT_ID --company-id $PAPERCLIP_COMPANY_ID"
  echo "  3. Ver logs: paperclipai logs --agent $AGENT_ID --company-id $PAPERCLIP_COMPANY_ID"
else
  echo "❌ Falha ao criar agente:"
  echo $RESPONSE | jq '.'
  exit 1
fi

# Limpar
rm -f /tmp/agent-payload.json
```

Execute:
```bash
chmod +x setup-agent.sh
./setup-agent.sh seu-api-key seu-company-id
```

---

## Comparação dos Métodos

| Aspecto | Dashboard | CLI | API REST |
|---------|-----------|-----|----------|
| **Facilidade** | ⭐⭐⭐ Fácil | ⭐⭐ Médio | ⭐ Técnico |
| **Velocidade** | Média | Rápida | Rápida |
| **Automação** | Manual | Scripts | Full CI/CD |
| **Debugging** | Visual | Logs | Logs + JSON |
| **Melhor para** | Primeira configuração, testes | Uso diário, automação | Deploy em massa, IaC |

---

## Configurações Avançadas

### Múltiplos Projetos

Para usar o mesmo repositório de skills em múltiplos projetos:

```json
{
  "adapterConfig": {
    "model": "MiniMax",
    "cwd": "/path/to/specific/project",
    "skillsDir": "/shared/path/to/sm-skills"
  }
}
```

### Skills Dinâmicas por Projeto

Configure `desiredSkills` específicas para cada tipo de projeto:

**Projeto NestJS:**
```json
["pwf-plan", "pwf-work", "nestjs-conventions", "typeorm-migrations"]
```

**Projeto Next.js:**
```json
["pwf-plan", "pwf-work", "nextjs-conventions", "prisma-migrations"]
```

**Projeto Angular:**
```json
["pwf-plan", "pwf-work", "angular-conventions"]
```

### Presets de Workflow

Use o campo `capabilities` para ativar presets:

```json
{
  "capabilities": "Development with preset: nestjs-api. Uses pwf-plan, pwf-work-plan, nestjs-conventions."
}
```

---

## Troubleshooting

### Skills não aparecem

1. Verifique se `skillsDir` aponta para a **raiz** do repositório (onde está `skills/`)
2. Confirme que cada skill tem arquivo `SKILL.md` com frontmatter YAML
3. Verifique logs: `paperclipai logs --agent <id> --verbose`

### Erro de autenticação

```bash
# Verifique se API key está configurada
echo $PAPERCLIP_API_KEY

# Teste conectividade
curl -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  $PAPERCLIP_API_URL/api/companies/$PAPERCLIP_COMPANY_ID/agents
```

### Modelo MiniMax não responde

1. Verifique se `model` está exatamente `"MiniMax"`
2. Confirme que o agente tem budget disponível
3. Verifique se o Paperclip está rodando: `paperclipai status`

---

## Fluxo Completo: Do Zero ao Agente Funcionando

```bash
# 1. Clone o repositório de skills
git clone https://github.com/sideminddev/sm-skills.git ~/sm-skills

# 2. Configure variáveis
export PAPERCLIP_API_KEY="sua-key"
export PAPERCLIP_COMPANY_ID="sua-company"
export PAPERCLIP_API_URL="http://localhost:3100/api"

# 3. Crie agente via CLI
paperclipai agent hire \
  --name "meu-dev" \
  --role "engineer" \
  --adapter-type opencode_local \
  --adapter-config "{\"model\": \"MiniMax\", \"cwd\": \"$PWD\", \"skillsDir\": \"$HOME/sm-skills\"}" \
  --desired-skills "pwf-plan,pwf-work,systematic-debugging" \
  --company-id $PAPERCLIP_COMPANY_ID

# 4. Liste agentes para pegar o ID
paperclipai agent list --company-id $PAPERCLIP_COMPANY_ID

# 5. Crie uma issue de teste no Paperclip dashboard

# 6. Atribua a issue ao agente e acorde
paperclipai agent wake <agent-id> --company-id $PAPERCLIP_COMPANY_ID
```

---

## Referências

- **Paperclip Docs:** https://docs.paperclip.ing
- **Adapters:** https://docs.paperclip.ing/adapters/overview
- **Skills:** https://docs.paperclip.ing/guides/agent-developer/writing-a-skill
- **API Reference:** `$PAPERCLIP_API_URL/docs` (Swagger/OpenAPI)

---

**Última atualização:** 2026-04-18  
**Versão do Guia:** 1.0
