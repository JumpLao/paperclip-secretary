# Paperclip Secretary Skill

An OpenClaw skill for managing Paperclip AI company operations as Board Secretary.

## What This Does

Automates Paperclip AI operations via API:
- **Company management** — Create and manage companies
- **Project setup** — Create projects with GitHub repos + workspaces
- **Goal alignment** — Set company/team/agent goals, link projects to goals
- **Issue delegation** — Create issues, assign to agents

## Chain of Command

```
Board (Human)
    ↓ Strategic decisions (WHAT)
Secretary (This agent)
    ↓ Execute via Paperclip API
CEO
    ↓ Operational decisions (HOW)
Team (CTO, Researcher, etc.)
```

## Features

### Proactive Workflow
- Detects new project ideas → asks if should use Paperclip
- Suggests company placement → Acme Labs, Acme Trading, etc.
- Suggests goal alignment → links projects to goals

### Goal-Driven
- Multiple goals per company (vision + quarterly + metrics)
- Project-level goals linked to company goals
- Goals fed to agents for context (assumed)

### Project Creation = Complete Setup
1. Creates GitHub repo (private)
2. Clones to local path
3. Creates Paperclip project with workspace
4. Links to relevant goals

## Requirements

- **Paperclip AI** running locally (or accessible via API)
- **GitHub CLI** (`gh`) authenticated
- **OpenClaw** with skills support

## Installation

```bash
# Clone to your OpenClaw skills directory
cd ~/.openclaw/workspace/skills
git clone https://github.com/your-org/paperclip-secretary.git
```

## Configuration

Add to your `TOOLS.md` (local notes):

```markdown
## Paperclip Companies

| Company | ID | Purpose |
|---------|-----|---------|
| YourCompany | `company-uuid` | Description |

**API:** `http://127.0.0.1:3100/api` (local_trusted)
**UI:** `http://127.0.0.1:3100`
```

## Usage

Just talk to your OpenClaw agent:

```
You: "อยากทำ trading bot"
Agent: "เก็บใน Paperclip ไหม?"
You: "ใช่"
Agent: "สร้างใน company ไหน? (Acme Trading, Acme Labs, etc.)
       Goal: 'Q2: Build profitable strategy'
       หรืออยากตั้ง goal ใหม่?"
```

## Paperclip Setup

Ensure Paperclip is in `local_trusted` mode:

```bash
# Check deployment mode
cat ~/.paperclip/instances/default/config.json | jq '.server.deploymentMode'
# Should return: "local_trusted"
```

## API Reference

See `references/api-reference.md` for full Paperclip API documentation.

## Philosophy

- **Board = WHAT, CEO = HOW** — Secretary executes, doesn't micromanage
- **Proactive > Reactive** — Suggest, don't just ask
- **Goals drive behavior** — Align everything to objectives
- **Source of truth** — Paperclip stores goals/projects/issues, not local files

## License

MIT

## Contributing

Contributions welcome! This skill is designed to be generic and shareable.
