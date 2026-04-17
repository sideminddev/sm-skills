---
name: git-operations
description: Use this skill whenever an agent needs to interact with Git repositories on GitHub. Triggers include cloning a repository, creating branches, committing and pushing changes, opening pull requests, and creating new repositories in an organization. Also use when the agent needs to determine which organization/repository to operate on from project context, or when it must ask the user (via issue comment) for the target repo. Always use this skill before running any git or GitHub CLI command.
source: sideminddev
tags:
  - git
  - github
  - version-control
  - pull-request
  - repository
tools:
  - paperclip
---

# Git Operations Skill

This skill teaches paperclipAI agents how to safely clone, develop, and push changes to
GitHub repositories using a Personal Access Token (`GITHUB_TOKEN`).

---

## 1. Context Discovery — Which repo to operate on?

Before touching any Git command, the agent **must** know:

| Variable | Where to get it |
|---|---|
| `GITHUB_ORG` | Project settings → "Set repo" (format: `org/repo`) |
| `GITHUB_REPO` | Same as above |
| `GITHUB_TOKEN` | Environment variable injected by paperclipAI |

### 1a. Token availability

Always verify the token is present before proceeding:

```bash
if [ -z "$GITHUB_TOKEN" ]; then
  echo "ERROR: GITHUB_TOKEN is not set. Cannot authenticate with GitHub."
  exit 1
fi
```

### 1b. Repo context from project settings

The paperclipAI project may have a repository configured via the **"Set repo"** function in project settings. This provides the repo in `org/repo` format. Parse it:

```bash
# Expected format: "organization/repository"
GITHUB_ORG=$(echo "$REPO_CONTEXT" | cut -d'/' -f1)
GITHUB_REPO=$(echo "$REPO_CONTEXT" | cut -d'/' -f2)
```

### 1c. No repo context — ask via issue comment

If the agent does **not** have a configured repository, it must ask by posting a comment on the current issue before doing anything else:

```
> 🤖 Não encontrei um repositório configurado para este projeto.
> Para continuar, por favor informe:
> - **Organização GitHub:** (ex: `minha-empresa`)
> - **Repositório:** (ex: `meu-projeto`)
>
> Ou configure em: Configurações do Projeto → "Set repo"
```

Stop all further execution until the user replies with the org/repo.

---

## 2. Authentication

All Git and GitHub API operations use token-based authentication. Never prompt for passwords.

### Configure Git to use the token

```bash
git config --global credential.helper store
echo "https://oauth2:${GITHUB_TOKEN}@github.com" > ~/.git-credentials
```

Or inline per-command (preferred for security):

```bash
REPO_URL="https://oauth2:${GITHUB_TOKEN}@github.com/${GITHUB_ORG}/${GITHUB_REPO}.git"
```

### GitHub CLI (gh) authentication

If `gh` is available:

```bash
echo "$GITHUB_TOKEN" | gh auth login --with-token
```

---

## 3. Cloning a Repository

```bash
git clone "https://oauth2:${GITHUB_TOKEN}@github.com/${GITHUB_ORG}/${GITHUB_REPO}.git" repo
cd repo
```

For a specific branch:

```bash
git clone --branch main --single-branch \
  "https://oauth2:${GITHUB_TOKEN}@github.com/${GITHUB_ORG}/${GITHUB_REPO}.git" repo
```

---

## 4. Branching Conventions

> 📖 Para referência rápida de comandos git (status, rebase, undo, stash, remotes), leia `references/git-cheatsheet.md`.

**Never commit directly to `main` or `master`.** Always create a feature branch.

```bash
# Name the branch after the issue or task
BRANCH_NAME="fix/issue-${ISSUE_NUMBER}-short-description"
# or
BRANCH_NAME="feat/issue-${ISSUE_NUMBER}-short-description"

git checkout -b "$BRANCH_NAME"
```

Naming pattern: `type/issue-NNN-kebab-description`
- `fix/` — bug fixes
- `feat/` — new features
- `chore/` — maintenance, dependencies
- `docs/` — documentation only

---

## 5. Making Changes and Committing

```bash
# Stage all changes
git add -A

# Or stage specific files
git add path/to/file.ext

# Commit with a conventional message
git commit -m "fix: correct null pointer in auth handler (closes #${ISSUE_NUMBER})"
```

Commit message format: `type: short description (closes #NNN)`

---

## 6. Pushing Changes

```bash
git push "https://oauth2:${GITHUB_TOKEN}@github.com/${GITHUB_ORG}/${GITHUB_REPO}.git" \
  "$BRANCH_NAME"
```

Or if the remote is already configured:

```bash
git remote set-url origin \
  "https://oauth2:${GITHUB_TOKEN}@github.com/${GITHUB_ORG}/${GITHUB_REPO}.git"
git push -u origin "$BRANCH_NAME"
```

---

## 7. Opening a Pull Request

> 📖 Para referência completa de todos os endpoints da API do GitHub (PRs, branches, issues, conteúdo de arquivos, actions), leia `references/github-api.md` antes de fazer chamadas à API REST diretamente.

Use the GitHub CLI:

```bash
gh pr create \
  --title "fix: short description of change" \
  --body "Closes #${ISSUE_NUMBER}

## Summary
- What was changed
- Why it was changed

## Testing
- Steps to verify the fix
" \
  --base main \
  --head "$BRANCH_NAME"
```

Or use the GitHub REST API directly:

```bash
curl -s -X POST \
  -H "Authorization: token ${GITHUB_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://api.github.com/repos/${GITHUB_ORG}/${GITHUB_REPO}/pulls" \
  -d "{
    \"title\": \"fix: short description\",
    \"body\": \"Closes #${ISSUE_NUMBER}\",
    \"head\": \"${BRANCH_NAME}\",
    \"base\": \"main\"
  }"
```

---

## 8. Creating a New Repository in an Organization

```bash
curl -s -X POST \
  -H "Authorization: token ${GITHUB_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://api.github.com/orgs/${GITHUB_ORG}/repos" \
  -d "{
    \"name\": \"${NEW_REPO_NAME}\",
    \"description\": \"${REPO_DESCRIPTION}\",
    \"private\": true,
    \"auto_init\": true,
    \"gitignore_template\": \"Node\"
  }"
```

Key fields:
- `"private": true` — default to private; change only if explicitly requested
- `"auto_init": true` — creates initial commit with README
- `"gitignore_template"` — optional, e.g., `"Node"`, `"Python"`, `"Go"`

After creation, clone and configure:

```bash
git clone "https://oauth2:${GITHUB_TOKEN}@github.com/${GITHUB_ORG}/${NEW_REPO_NAME}.git"
```

---

## 9. Posting Comments on Issues

Use this to communicate status back to the user mid-task:

```bash
curl -s -X POST \
  -H "Authorization: token ${GITHUB_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://api.github.com/repos/${GITHUB_ORG}/${GITHUB_REPO}/issues/${ISSUE_NUMBER}/comments" \
  -d "{\"body\": \"✅ Branch \`${BRANCH_NAME}\` criada e PR aberto: ${PR_URL}\"}"
```

---

## 10. Checking Repository Info

Verify the repo exists and the token has access:

```bash
curl -s \
  -H "Authorization: token ${GITHUB_TOKEN}" \
  "https://api.github.com/repos/${GITHUB_ORG}/${GITHUB_REPO}" \
  | jq '.full_name, .default_branch, .permissions'
```

---

## 11. Error Handling

| HTTP Status | Meaning | Action |
|---|---|---|
| `401` | Token invalid or expired | Post comment asking user to renew `GITHUB_TOKEN` |
| `403` | Insufficient permissions | Post comment listing required scopes (see below) |
| `404` | Repo not found | Post comment asking user to confirm org/repo name |
| `422` | Branch already exists or PR already open | Use existing branch / link existing PR |

### Required token scopes

The `GITHUB_TOKEN` must have:
- `repo` — full repository access (read + write + PRs)
- `workflow` — if the task triggers GitHub Actions
- `admin:org` → `write:org` — only if creating org-level repos

---

## 12. Safe Defaults Checklist

Before finishing any task, verify:

- [ ] Never committed to `main`/`master` directly
- [ ] Branch name follows `type/issue-NNN-description` pattern
- [ ] All commits reference the issue number
- [ ] PR is opened against the correct base branch
- [ ] No secrets or tokens appear in committed files
- [ ] `.env` and credential files are in `.gitignore`

---

## 13. Full Workflow Example

```bash
# 1. Setup
export GITHUB_ORG="minha-empresa"
export GITHUB_REPO="meu-projeto"
export ISSUE_NUMBER="42"
export BRANCH_NAME="fix/issue-42-corrige-auth"

# 2. Clone
git clone "https://oauth2:${GITHUB_TOKEN}@github.com/${GITHUB_ORG}/${GITHUB_REPO}.git" repo
cd repo

# 3. Branch
git checkout -b "$BRANCH_NAME"

# 4. Make changes (agent does work here)
echo "fix" >> README.md

# 5. Commit
git add -A
git commit -m "fix: corrige autenticação (closes #${ISSUE_NUMBER})"

# 6. Push
git remote set-url origin \
  "https://oauth2:${GITHUB_TOKEN}@github.com/${GITHUB_ORG}/${GITHUB_REPO}.git"
git push -u origin "$BRANCH_NAME"

# 7. Open PR via gh CLI
gh pr create \
  --title "fix: corrige autenticação" \
  --body "Closes #${ISSUE_NUMBER}" \
  --base main \
  --head "$BRANCH_NAME"
```

---

---

## Reference files

Leia os arquivos abaixo quando precisar de detalhes adicionais:

- `references/github-api.md` — **Leia quando for usar a API REST do GitHub diretamente** (criar repo, listar PRs, postar comentários, ler/escrever arquivos via API, verificar rate limit)
- `references/git-cheatsheet.md` — **Leia quando precisar de comandos git específicos** (rebase, stash, undo, inspecionar histórico, resolver conflitos)
