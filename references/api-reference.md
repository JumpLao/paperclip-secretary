# Paperclip API Reference

Base URL: `http://localhost:3100/api`

## Companies

### List Companies
```
GET /api/companies
```

### Get Company
```
GET /api/companies/{companyId}
```

### Create Company
```
POST /api/companies
{
  "name": "Company Name",
  "description": "Description"
}
```

### Update Company
```
PATCH /api/companies/{companyId}
{
  "name": "New Name",
  "description": "New description",
  "budgetMonthlyCents": 100000
}
```

### Archive Company
```
POST /api/companies/{companyId}/archive
```

---

## Goals

### List Goals
```
GET /api/companies/{companyId}/goals
```

### Get Goal
```
GET /api/goals/{goalId}
```

### Create Goal
```
POST /api/companies/{companyId}/goals
{
  "title": "Goal Title",
  "description": "Description",
  "level": "company",  // company | team | agent
  "status": "active"   // active | completed | paused
}
```

### Update Goal
```
PATCH /api/goals/{goalId}
{
  "status": "completed"
}
```

---

## Projects

### List Projects
```
GET /api/companies/{companyId}/projects
```

### Get Project
```
GET /api/projects/{projectId}
```

### Create Project (with workspace)
```
POST /api/companies/{companyId}/projects
{
  "name": "Project Name",
  "description": "Description",
  "goalIds": ["{goalId}"],  // optional
  "status": "planned",      // planned | in_progress | paused | completed | archived
  "workspace": {            // optional
    "name": "repo-name",
    "cwd": "/home/user/projects/repo-name",
    "repoUrl": "https://github.com/your-org/repo-name",
    "repoRef": "main",
    "isPrimary": true
  }
}
```

### Update Project
```
PATCH /api/projects/{projectId}
{
  "status": "in_progress"
}
```

---

## Workspaces

### List Workspaces
```
GET /api/projects/{projectId}/workspaces
```

### Add Workspace
```
POST /api/projects/{projectId}/workspaces
{
  "name": "repo-name",
  "cwd": "/path/to/workspace",
  "repoUrl": "https://github.com/org/repo",
  "repoRef": "main",
  "isPrimary": true
}
```

### Update Workspace
```
PATCH /api/projects/{projectId}/workspaces/{workspaceId}
```

### Delete Workspace
```
DELETE /api/projects/{projectId}/workspaces/{workspaceId}
```

---

## Agents

### List Agents
```
GET /api/companies/{companyId}/agents
```

### Get Agent
```
GET /api/agents/{agentId}
```

### Hire Agent
```
POST /api/companies/{companyId}/agents
{
  "name": "Agent Name",
  "role": "ceo",  // ceo | cto | developer | researcher | designer | etc.
  "instructions": "Agent instructions...",
  "heartbeatIntervalMs": 1800000  // optional, default 30 min
}
```

### Update Agent
```
PATCH /api/agents/{agentId}
{
  "instructions": "Updated instructions"
}
```

### Pause/Resume Agent
```
POST /api/agents/{agentId}/pause
POST /api/agents/{agentId}/resume
```

### Fire Agent
```
DELETE /api/agents/{agentId}
```

---

## Issues

### List Issues
```
GET /api/projects/{projectId}/issues
GET /api/projects/{projectId}/issues?status=open
GET /api/projects/{projectId}/issues?assigneeId={agentId}
```

### Get Issue
```
GET /api/issues/{issueId}
```

### Create Issue
```
POST /api/projects/{projectId}/issues
{
  "title": "Issue Title",
  "description": "Issue description",
  "priority": "medium",  // low | medium | high | urgent
  "assigneeIds": ["{agentId}"]  // optional
}
```

### Update Issue
```
PATCH /api/issues/{issueId}
{
  "status": "in_progress",  // backlog | todo | in_progress | review | done | cancelled
  "assigneeId": "{agentId}"
}
```

### Assign Issue
```
PATCH /api/issues/{issueId}
{
  "assigneeId": "{agentId}"
}
```

### Add Comment
```
POST /api/issues/{issueId}/comments
{
  "body": "Comment text"
}
```

---

## Dashboard

### Company Dashboard
```
GET /api/companies/{companyId}/dashboard
```

Returns summary: agents, projects, issues, activity

---

## Common Patterns

### Full Project Setup

```bash
# 1. Create GitHub repo
gh repo create your-org/my-project --private --clone /home/user/projects/my-project

# 2. Initialize git
cd /home/user/projects/my-project
git commit --allow-empty -m "Initial commit"
git push -u origin main

# 3. Create Paperclip project
curl -X POST http://localhost:3100/api/companies/{companyId}/projects \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Project",
    "description": "Project description",
    "status": "planned",
    "workspace": {
      "name": "my-project",
      "cwd": "/home/user/projects/my-project",
      "repoUrl": "https://github.com/your-org/my-project",
      "repoRef": "main",
      "isPrimary": true
    }
  }'
```

### Delegate Board Directive

```bash
# 1. Create issue
ISSUE_ID=$(curl -s -X POST http://localhost:3100/api/projects/{projectId}/issues \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Board: Research trading strategies",
    "description": "Board requests research on ML trading and SMC concepts",
    "priority": "high"
  }' | jq -r '.id')

# 2. Assign to CEO
curl -X PATCH http://localhost:3100/api/issues/$ISSUE_ID \
  -H "Content-Type: application/json" \
  -d '{ "assigneeId": "{ceo-agent-id}" }'
```
