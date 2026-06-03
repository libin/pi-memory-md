---
name: memory-write
description: Create or update pi-memory-md memory files using the native write/edit tools plus the bundled template script. Use whenever writing, creating, or updating memory files.
---

# Memory Write

Use this skill to safely create or update pi-memory-md memory files while preserving valid frontmatter.

## Workflow

### 1. Find the memory directory

Use [scripts/memory-write.sh](scripts/memory-write.sh) to resolve the project memory directory. Use the printed path as `<memory-dir>`.

**Critical:** DO NOT CREATE, UPDATE, or WRITE any memory file until `<memory-dir>` has been resolved and verified by [scripts/memory-write.sh](scripts/memory-write.sh).

### 2. Create a new memory file

Before creating a memory file, infer a proposed relative path, description, and tags from the user's request, then ask the user to confirm them unless they already provided these values explicitly.

Use [scripts/memory-write.sh](scripts/memory-write.sh) to create the file template. The script prints the created absolute file path. Read or edit that file next.

When the memory records a unit of work with an outcome (a fix, a decision, an attempt), pass an optional `status` as the last `create` argument so the model can later tell what already happened:

```
memory-write.sh create <memory-dir> core/project/<topic>.md "<description>" "tag1,tag2" <status>
```

### 3. Update an existing memory file

1. Use `read` on the existing file.
2. Use `edit` for targeted body changes when possible.
3. Preserve existing YAML frontmatter.
4. Refresh `updated` with [scripts/memory-write.sh](scripts/memory-write.sh).
5. When the outcome of the recorded work changes, set its `status` (this also refreshes `updated`):

```
memory-write.sh set-status <memory-dir> <relative-path> <status>
```

If a full rewrite is necessary, include the complete frontmatter and body in native `write`, then refresh `updated`.

## Status — record the outcome, not just "done"

`status` is optional but valuable: it lets future sessions distinguish work that already
succeeded from approaches that failed, so they neither silently redo finished work nor retry a
dead end. Allowed values:

| status | meaning |
|--------|---------|
| `verified` | done and confirmed working |
| `done` | done, not independently confirmed |
| `in-progress` | started, not finished |
| `failed` | tried, did not work (kept so it is not repeated) |
| `superseded` | replaced by a newer approach |

The delivered memory index surfaces `status` next to each file. Before redoing work, check
memory: never silently repeat work marked `done`/`verified`, or retry an approach marked
`failed` — surface it and ask the user. Prefer stable references in the body (a file plus a
function/symbol name) over line numbers, which rot on refactor.

## Placement rules

- Put always-needed context under `core/`.
- Put project-specific auto-delivered memories under `core/project/`.
- Use root-level folders like `docs/`, `archive/`, `research/`, or `references/` for non-core references.
- Never create a root-level `project/` folder; use `core/project/`.

## Frontmatter shape

The script creates:

```yaml
---
description: "Human-readable description"
tags:
  - "tag"
status: "verified"   # optional: verified | done | in-progress | failed | superseded
created: "YYYY-MM-DD"
updated: "YYYY-MM-DD"
---
```

`status` is omitted unless you pass it to `create`/`set-status`; existing files without it are unaffected.
