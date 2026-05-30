---
name: memory-init
description: Initialize memory repository - clone git repo and create directory structure. Use when you need to set up pi-memory-md for the first time or initalize project's memory files.
---

## Overview

1. Run [scripts/memory-init.sh](scripts/memory-init.sh) to clone/sync repo and create directories
2. Read and copy template files from [templates/](templates/) (user decides which)

> **Resolve paths against this skill's directory, not the current working directory.**
> `scripts/memory-init.sh` and `templates/` mean `<dir of this SKILL.md>/scripts/memory-init.sh`
> and `<dir of this SKILL.md>/templates/`. Find the skill directory first, then use the
> absolute path. Do **not** run `bash scripts/memory-init.sh` from a random cwd.
>
> **These files ship with the skill. If the script or templates are missing, STOP and report
> the absolute path you searched. Never recreate `scripts/` or `templates/` yourself, and never
> write them into the extension source checkout.** Recreating them produces broken duplicates.

## TL;DR (happy path)

```bash
SKILL_DIR="<absolute dir containing this SKILL.md>"
bash "$SKILL_DIR/scripts/memory-init.sh"   # idempotent; reads settings, creates dirs
```

Then optionally copy templates (Step 3), then verify with the `memory_check` tool. The script is
plain bash and needs no model reasoning — run it as-is. Everything after it is optional.

## Prerequisites

Before running this skill, ensure:
- Package installed: `pi install npm:pi-memory-md`
- `pi-memory-md` configured in your settings file (see schema below). At minimum `localPath`
  must be set; `repoUrl` is required **only** if you want git clone/sync.
- If using git sync: the repo exists, you can push to it, and this machine has a working
  GitHub credential (SSH key added to your account, or an HTTPS token). No credential =
  clone/sync will fail with `Permission denied (publickey)`; use a local-only store instead
  (omit `repoUrl`).

### Settings schema

Global settings live in `~/.pi/agent/settings.json` (i.e. `$PI_CODING_AGENT_DIR/settings.json`),
or project-level `./.pi/settings.json`. The block is keyed by `pi-memory-md`:

```json
{
  "pi-memory-md": {
    "enabled": true,
    "autoSync": { "onSessionStart": true },
    "memoryDir": {
      "repoUrl": "git@github.com:<you>/<your-memory-repo>.git",
      "localPath": "~/.pi/memory-md",
      "globalMemory": "global"
    }
  }
}
```

> **Three things that break setups — check before writing settings:**
> - The settings file is `~/.pi/agent/settings.json`, **not** `~/.pi/settings.json` (the latter is ignored).
> - `localPath` is a **dedicated memory store** (e.g. `~/.pi/memory-md`). It is **never** the
>   extension source checkout (the directory containing this package's `package.json`).
> - `repoUrl` is a repo **you** can push to (your own fork/memory repo), not the upstream project.
>   Omit `repoUrl` entirely for a local-only store with no git.
>
> For a local-only store (no git), set `localPath` + optional `globalMemory`, omit `repoUrl`,
> and skip the clone — the script creates the directories directly.

## Execution Steps

### Step 1: Run Initialization Script

Execute the initialization script at `<skill dir>/scripts/memory-init.sh` (use the absolute
path; see the path note in Overview). The script is idempotent and safe to re-run.

If the script is not found, **STOP** — report the absolute path searched and do not recreate it.

The script will:
1. Read settings from `.pi/settings.json` or `$PI_CODING_AGENT_DIR/settings.json`
2. Calculate memory directories
3. Refuse to run if `localPath` points at the extension source checkout
4. Clone or sync the git repository (only when `repoUrl` is set)
5. Create `core/project/`

### Step 2: Configure globalMemory (if applicable)

Read settings from `.pi/settings.json` or `$PI_CODING_AGENT_DIR/settings.json` and check for `globalMemory` configuration.

Then ask user whether they also want to create default global files under the configured `globalMemory` directory:
- `{globalMemory}/USER.md` from `user-template.md`
- `{globalMemory}/MEMORY.md` from `memory-template.md`
- `{globalMemory}/TASK.md` from `task-template.md`

### Step 3: Copy Template Files for Project Memory (Optional)

Ask user which project templates to create in [templates/](templates/):

```
Which project template files would you like to create? (select all that apply)
1. task-template.md - Project tasks and planning template
2. user-template.md - Project user profile and preferences template
3. None (skip project templates)
```

If user selects templates, copy them from `templates/` to the target paths:

```bash
cp templates/task-template.md {projectMemoryDir}/core/TASK.md
cp templates/user-template.md {projectMemoryDir}/core/USER.md
```

### Step 4: Import Preferences from AGENTS.md (Optional)

This step extracts preferences from AGENTS.md to populate project `core/USER.md` and, if global memory is enabled, `{globalMemory}/USER.md`.

1. **Find AGENTS.md** (check in order):
   - Project root: `{cwd}/AGENTS.md`
   - Project: `{cwd}/.pi/agent/AGENTS.md`
   - Global: `~/.pi/agent/AGENTS.md`

2. **Ask user**: Do you want to import preferences from AGENTS.md?
   - If NO, skip to "Summarize and confirm"
   - If YES, continue

3. **Read AGENTS.md** and extract relevant sections:
   - IMPORTANT Rules
   - Code Quality Principles
   - Coding Style Preferences
   - Architecture Principles
   - Development Workflow
   - Technical Preferences

4. **Summarize and confirm**:
   ```
   Found these preferences in AGENTS.md:
   - IMPORTANT Rules: [1-2 sentence summary]
   - Code Quality Principles: [1-2 sentence summary]
   - Coding Style: [1-2 sentence summary]

   Include these in project core/USER.md and, if available, {globalMemory}/USER.md? (yes/no)
   ```

5. **If confirmed**, update or create the target profile files with:
   - `core/USER.md`
   - `{globalMemory}/USER.md` if global memory is enabled
   - Extracted content from AGENTS.md
   - Keep the existing frontmatter (description, tags, created)

6. **Ask for additional preferences**:
   ```
   Any additional preferences to add to USER.md? (e.g., communication style, specific tools)
   ```

### Step 5: Create Additional Folders (Optional)

Ask user whether they want to create any additional folders beyond `core/project`.

Examples:
- `reference/`
- `archive/`
- Any custom project-specific folder

If YES, ask for the folder names and create them under the project memory directory.

### Step 6: Verify Setup

Call `memory_check` tool to verify setup is correct.

## Memory Repository Structure

```
{localPath}/
├── {globalMemory}/            # (if globalMemory config block exists)
│   ├── USER.md                # Shared user profile and preferences
│   ├── MEMORY.md              # Shared durable notes, conventions, and lessons learned
│   └── TASK.md                # Shared task and planning file
└── {project-name}/
    └── core/
        ├── USER.md            # Project user profile and preferences
        ├── project/           # Project memory files
        └── TASK.md            # Task and planning file
```

## Workflow Guide

```
START
  │
  ▼
Run scripts/memory-init.sh
  │
  ▼
Script reads settings, clones/syncs repo, and creates project directories
  │
  ▼
Check script result: globalMemory enabled?
  │
  ├─ NO ──► Continue with project setup
  │
  └─ YES
      │
      ▼
  Ensure global memory directory exists
      │
      ▼
  Ask: Create {globalMemory}/USER.md, {globalMemory}/MEMORY.md, and {globalMemory}/TASK.md?
      │
      ├─ NO ──► Skip global files
      │
      └─ YES
          │
          ▼
      Copy user-template.md to {globalMemory}/USER.md
          │
          ▼
      Copy memory-template.md to {globalMemory}/MEMORY.md
          │
          ▼
      Copy task-template.md to {globalMemory}/TASK.md
          │
          ▼
Continue with project setup
  │
  ▼
Ask: Which project templates to create?
  │
  ├─ None ──► Skip templates
  │
  └─ Select templates
      │
      ▼
  Copy selected project templates
      │
      ▼
  Never create project core/MEMORY.md in this flow
      │
      ▼
Ask: Import preferences from AGENTS.md?
      │
  ├─ NO ──► Skip import
  │
  └─ YES
      │
      ▼
  Read AGENTS.md and extract preferences
      │
      ▼
  Ask: Confirm import to project core/USER.md and, if available, {globalMemory}/USER.md?
      │
      ├─ NO ──► Ask for additional preferences
      │
      └─ YES
          │
          ▼
      Update project core/USER.md and, if available, {globalMemory}/USER.md
          │
          ▼
      Ask: Additional preferences?
          │
          ▼
Ask: Create any additional folders?
  │
  ▼
Verify with /memory-check
  │
  ▼
DONE
```

## Error Handling

| Error | Solution |
|-------|----------|
| `settings not found` | Configure `pi-memory-md` in settings file |
| `repoUrl not configured` | Add `repoUrl` to settings |
| `Permission denied` | Check SSH keys: `ssh -T git@github.com` |
| `Directory exists but not git` | Remove directory manually and retry |
| `Connection timeout` | Check network, try again |

## Templates

Copy these templates to start:

- [templates/task-template.md](templates/task-template.md) — Project tasks and planning template
- [templates/user-template.md](templates/user-template.md) — User profile and preferences template
- [templates/memory-template.md](templates/memory-template.md) — Hermes-inspired durable notes, conventions, and lessons learned template for `globalMemory` only

## Scripts

- [scripts/memory-init.sh](scripts/memory-init.sh) — Initialize memory repository (clone repo, create minimal directories)

## Related Skills

- `memory-management` - Create and manage memory files
- `memory-sync` - Git synchronization
- `memory-search` - Find information in memory
