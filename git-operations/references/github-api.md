# GitHub REST API — Reference for paperclipAI Agents

Base URL: `https://api.github.com`

All requests require:
```
Authorization: token ${GITHUB_TOKEN}
Content-Type: application/json
```

---

## Repositories

### Get repo info
```bash
GET /repos/{org}/{repo}
```

### List org repos
```bash
GET /orgs/{org}/repos?type=all&per_page=100
```

### Create org repo
```bash
POST /orgs/{org}/repos
{
  "name": "repo-name",
  "description": "...",
  "private": true,
  "auto_init": true
}
```

### Delete repo (use with extreme caution)
```bash
DELETE /repos/{org}/{repo}
```

---

## Branches

### List branches
```bash
GET /repos/{org}/{repo}/branches
```

### Get branch
```bash
GET /repos/{org}/{repo}/branches/{branch}
```

### Create branch via API
```bash
POST /repos/{org}/{repo}/git/refs
{
  "ref": "refs/heads/feat/new-branch",
  "sha": "<commit-sha-of-base>"
}
```

---

## Pull Requests

### List PRs
```bash
GET /repos/{org}/{repo}/pulls?state=open
```

### Create PR
```bash
POST /repos/{org}/{repo}/pulls
{
  "title": "PR title",
  "body": "Description",
  "head": "feature-branch",
  "base": "main"
}
```

### Get PR
```bash
GET /repos/{org}/{repo}/pulls/{pull_number}
```

### Merge PR
```bash
PUT /repos/{org}/{repo}/pulls/{pull_number}/merge
{
  "merge_method": "squash",
  "commit_title": "feat: merged feature (#42)"
}
```

---

## Issues & Comments

### List issues
```bash
GET /repos/{org}/{repo}/issues?state=open
```

### Get issue
```bash
GET /repos/{org}/{repo}/issues/{issue_number}
```

### Post comment on issue
```bash
POST /repos/{org}/{repo}/issues/{issue_number}/comments
{
  "body": "Comment text here"
}
```

### List issue comments
```bash
GET /repos/{org}/{repo}/issues/{issue_number}/comments
```

---

## Contents (file read/write via API)

### Get file content
```bash
GET /repos/{org}/{repo}/contents/{path}?ref=main
```
Returns base64-encoded content. Decode with: `echo "$content" | base64 -d`

### Create or update file
```bash
PUT /repos/{org}/{repo}/contents/{path}
{
  "message": "commit message",
  "content": "<base64-encoded-content>",
  "sha": "<existing-file-sha-if-updating>",
  "branch": "feature-branch"
}
```

---

## Actions / Workflows

### List workflows
```bash
GET /repos/{org}/{repo}/actions/workflows
```

### Trigger workflow dispatch
```bash
POST /repos/{org}/{repo}/actions/workflows/{workflow_id}/dispatches
{
  "ref": "main",
  "inputs": {}
}
```

---

## User / Token Verification

### Verify token identity
```bash
GET /user
```

### Check rate limit
```bash
GET /rate_limit
```

---

## Pagination

GitHub paginates results. To get all pages:

```bash
page=1
while true; do
  result=$(curl -s \
    -H "Authorization: token ${GITHUB_TOKEN}" \
    "https://api.github.com/orgs/${GITHUB_ORG}/repos?per_page=100&page=${page}")
  
  count=$(echo "$result" | jq 'length')
  [ "$count" -eq 0 ] && break
  
  echo "$result" | jq '.[].full_name'
  ((page++))
done
```
