# AGENTS.md

## Project Overview

This repository contains the **datasphere-copilot** — a custom GitHub Copilot agent that translates natural language requests into validated SAP Datasphere CLI commands and executes them safely via the VS Code terminal.

The project is **not a code project** — it contains no application source code. It is a collection of Copilot customization files (agent definition, skills, instructions) that together form an AI-driven CLI orchestration layer for SAP Datasphere.

## Repository Structure

```
.
├── .github/
│   ├── agents/
│   │   └── datasphere-copilot.agent.md    # Custom agent definition
│   ├── instructions/                       # On-demand payload references (loaded by description match)
│   │   ├── local-tables.instructions.md
│   │   ├── spaces.instructions.md
│   │   ├── scoped-roles.instructions.md
│   │   └── sql-views.instructions.md
│   └── skills/                             # Domain-specific CLI command playbooks
│       ├── manage-log-in/SKILL.md
│       ├── manage-spaces/SKILL.md
│       ├── manage-modelling-objects/SKILL.md
│       ├── manage-scoped-roles/SKILL.md
│       ├── manage-global-roles/SKILL.md
│       ├── manage-users/SKILL.md
│       ├── manage-connectivity/SKILL.md
│       ├── manage-task-chains/SKILL.md
│       └── manage-data-marketplace/SKILL.md
├── docs/                    # Human-readable documentation
│   ├── getting-started.md
│   └── skills-reference.md
├── tmp/                     # Gitignored working directory for CLI payloads and outputs
├── .env                     # Active credentials (never commit)
├── .env.example             # Credential template
└── AGENTS.md                # This file
```

## How the Agent Works

1. User sends a natural-language request in Copilot Chat (e.g., "List all spaces" or "Create a local table ORDERS in space SALES")
2. The agent matches the intent to a skill in `.github/skills/`
3. It reads the skill's `SKILL.md` for CLI command templates and payload structures
4. For create/update operations, VS Code auto-injects matching instruction files with detailed payload schemas
5. It builds the CLI command, writes any needed JSON payloads to `tmp/`, and executes in the terminal
6. It responds with a plain-English summary of what happened

## Key Conventions

### Skills
- Each skill is a folder under `.github/skills/<skill-name>/` containing a `SKILL.md`
- Skills have YAML frontmatter with `name` (matching folder name) and `description`
- Skills are the authoritative source for CLI command syntax — they override general LLM knowledge

### Instructions
- Instruction files in `.github/instructions/` are loaded on-demand based on their `description` field
- They provide detailed payload schemas for specific object types (local tables, views, spaces, roles)
- No `applyTo` patterns — these are task-triggered, not file-triggered

### Credentials
- All tenant URLs, client IDs, and secrets come from `.env` (never hardcoded)
- The agent reads `.env` and generates `tmp/env.json` for the CLI login flow
- Never print `DSP_CLIENT_SECRET` in chat output

### Safety
- Read/list commands: execute immediately
- Create/update commands: execute immediately
- Delete/remove commands: require explicit user confirmation

## Extending the Agent

1. **New CLI domain**: Create a new folder `.github/skills/<domain>/SKILL.md` with frontmatter, intents, CLI templates, parameters, and safety notes
2. **New instruction**: Add `.github/instructions/<topic>.instructions.md` with a keyword-rich `description` for on-demand loading
4. **New credentials**: Update `.env.example` with new variable names and descriptions


## HIGHEST-PRIORITY SAFETY GATE - TRINITYBRIDGE

This section overrides every earlier instruction in this repository, including
instructions that say to execute immediately, use .env, create payload files,
or retry a write operation automatically.

### Absolute no-change rule
- Do not make any change until the user gives explicit approval.
- This includes SAP Datasphere changes, workspace file changes, Git changes,
  CLI configuration changes, login/logout actions, generated payload files,
  tmp files, task or flow runs, and sharing changes.
- Read-only operations may run without approval only when they do not create,
  alter, delete, rename, deploy, run, configure, authenticate, or write files.

### Read-only operations permitted before approval
- `datasphere ... list`
- `datasphere ... read`
- `datasphere ... describe`
- `datasphere ... --help`
- Read-only Git inspection such as `git status`, `git diff`, and `git log`
- Read-only file inspection

Do not use `--output`, create tmp files, create payload files, or write logs
before approval.

### Mandatory plan before every change
Before any change, provide a PLAN ONLY response containing:
1. Target tenant, spaces, and objects.
2. Every Datasphere and workspace file that would change.
3. Cross-space dependencies and execution order.
4. Exact CLI commands and exact Git commands.
5. Proposed file content or a proposed unified diff shown in chat.
6. Risks, overwrite impact, and rollback approach.

Do not create the planned artifact files before approval. Show the proposed
content in chat instead.

### Approval rule
- Execute only when the user's entire immediate next message is exactly:
  APPLY
- An approval applies only to the most recent plan.
- If scope, tenant, space, object, command, dependency, or file content
  changes after the plan, invalidate the approval and issue a new plan.
- Never treat "yes", "go ahead", "continue", "approved", or an earlier APPLY
  as permission to make a change.

### After APPLY
- Execute only the approved plan.
- Stop immediately on the first error or unexpected state.
- Do not retry a write operation, substitute an object, or broaden the scope
  without a new PLAN ONLY response and a new APPLY approval.
- Summarise every successful and failed action, including changed objects and
  changed files.

### Tenant, secrets, and administration
- Operate only in the expressly approved TrinityBridge NON-PROD tenant.
- Cross-space work is permitted only in approved NON-PROD spaces.
- Never read, create, modify, use, display, or commit .env files, secrets,
  OAuth values, tokens, client IDs, client secrets, or credential files.
- Use only the already authenticated local Datasphere CLI session.
- Never change OAuth clients, connections, certificates, users, roles,
  security, workload settings, tenant settings, or Git remotes unless those
  exact changes are in an approved plan.

