# Copilot Agent Instructions for Datasphere CLI

## Purpose
This workspace contains the `datasphere-copilot` agent, which translates natural language requests into validated SAP Datasphere CLI commands and executes them safely.

## Workspace Layout
| Path | Purpose |
|---|---|
| `.github/agents/datasphere-copilot.agent.md` | Custom agent definition |
| `.github/instructions/*.instructions.md` | Object-type-specific payload references (loaded on-demand) |
| `.github/skills/<name>/SKILL.md` | Domain-specific CLI command playbooks |
| `docs/` | Reference documentation for each command area |
| `.env` | Active credentials (never commit) |
| `.env.example` | Credential template to share with team |

## Rules That Apply to All Agents in This Workspace

### Skills Are Authoritative
- Before generating any Datasphere CLI command, load the matching `.github/skills/<name>/SKILL.md` file.
- Skill command templates override any general LLM knowledge about CLI flags.
- If no matching skill exists, proceed using built-in CLI knowledge — never block.

### Credentials Come From .env Only
- All tenant URLs, client IDs, and secrets must be read from `.env`.
- Never hardcode or guess credential values.
- If `.env` is missing or incomplete, prompt the user to fill it in from `.env.example`.

### Output Style
- After executing any command, respond in plain English only (e.g. "Space X created with 10 GB storage.").
- Do NOT paste raw JSON or CLI output unless the user explicitly asks for it or the command failed and the raw error is required to diagnose the problem.

### Safety
- List/read/describe commands: execute immediately.
- Create/update commands: execute immediately.
- Delete/remove commands: require explicit "yes" or "confirm" before executing.
- Never print `DSP_CLIENT_SECRET` in chat output.
- Never ask the user to copy and run a command manually. The agent always runs commands itself in the terminal, including any follow-up or analysis commands.

## Extending the Agent
- Add new `.github/skills/<domain>/SKILL.md` folders to support new command areas.
- Mirror the structure of existing skill files: YAML frontmatter, intents, CLI templates, parameters table, safety notes.
- Add matching `.github/instructions/<topic>.instructions.md` files for detailed payload schemas.
- Update `.env.example` if new environment variables are needed.

## Example Prompts
- "Log in to the dev tenant"
- "List all spaces"
- "Create a local table XDST in space DENIS with columns ID INTEGER, NAME NVARCHAR(100)"
- "Show me the command before you run it"
- "Delete space TESTSPACE"

## TrinityBridge Local Safety Override

These rules override any earlier conflicting instructions.

### Environment boundary
- Operate only against the expressly approved TrinityBridge SAP Datasphere NON-PROD / DEV tenant.
- Never operate against UAT, Test, Production, or any tenant not explicitly approved by the user.
- Never assume a target space. If a space is not specified, prepare a plan only.

### Approval boundary
- Read, list, and describe commands may run immediately.
- For every create, update, deploy, rename, delete, task-chain run, connection,
  certificate, user, role, space, workload, or security action:
  1. Inspect the current state first.
  2. Prepare the proposed artifact or payload in the workspace.
  3. Show the object name, target space, dependencies, exact CLI command, and Git diff.
  4. Do not execute until the user replies exactly: APPLY.
- Never delete or overwrite an object unless the user explicitly confirms the
  technical object name and target space.

### Credentials and authentication
- Never create, read, modify, or use .env files.
- Never create, read, modify, or use secrets files, token files, or login-options files in the workspace.
- Use only an already authenticated local SAP Datasphere CLI session.
- Never display, log, copy, commit, or expose client IDs, client secrets, access tokens,
  refresh tokens, OAuth URLs, or the contents of any credential file.
- Never run: datasphere config secrets show

### TrinityBridge conventions
- 1LT_* = inbound objects
- 2LT_* = modelling-layer tables
- 2GV_* = graphical views
- 2SV_* = SQL views
- 3AM_* = analytical models
- Store reusable artifact definitions under:
  artifacts/<space>/<object-type>/
- Use tmp/ only for transient, non-secret files.
## TrinityBridge NON-PROD Cross-Space Development Policy

These rules override any earlier restrictions that limit write activity to a single space.

### Approved tenant boundary
- Work only in the approved TrinityBridge SAP Datasphere NON-PROD tenant.
- Never operate against Production or any other tenant unless the user explicitly authorises it.

### Approved spaces
- Read and write work may be performed across these NON-PROD spaces when required:
  - 1_INBOUND_HIST_MIGR
  - 1_INBOUND_HIST_RAW
  - 1_INBOUND_S4_CURR
  - INBOUND
  - MODELLING
  - REPORTING
  - SAP_CONTENT
- Operate only where the authenticated user has the required Datasphere privileges.

### Cross-space change procedure
For any create, update, deployment, rename, delete, task, flow, or other write action:
1. Inspect the relevant source and target spaces first.
2. Identify all cross-space dependencies and the required execution sequence.
3. Prepare the artifact or payload files in the workspace.
4. Show:
   - every affected space
   - affected object names and types
   - dependencies and deployment order
   - exact Datasphere CLI commands
   - Git diff
5. Do not execute any write action until the user replies exactly: APPLY.

### Protected administration
- Never change OAuth clients, connections, certificates, users, roles, security,
  workload settings, or tenant administration without separate explicit approval.
- For SAP_CONTENT, never overwrite, delete, or alter SAP-delivered content unless
  the user explicitly names the object and confirms the intended change.

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

