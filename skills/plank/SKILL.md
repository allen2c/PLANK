---
name: plank
description: >-
  Manage project issues using the PLANK standard (Plaintext Local Agile Notes
  & Kanban). Use when creating issues, updating ticket status, managing epics,
  or setting up a new PLANK-compliant project in a Git repository.
---

# PLANK — AI Agent Workflow

Canonical spec: [PLANK.md](../../PLANK.md)

## Principles

- Files are the database. Git is the audit log.
- YAML frontmatter is the schema. Human narrative lives below.
- Define the minimum shared contract. Everything else is the project's choice.
- If a tool can't read it with `grep` and a YAML parser, it's too complex.

## Setup

A PLANK project needs one directory:

```plaintext
<repo-root>/
└── .plank/
    ├── issues/             # flat issue store — source of truth
    │   ├── PROJ-001.md
    │   └── PROJ-002.md
    └── epics/              # optional
        └── EP-01-auth.md
```

If `.plank/issues/` does not exist, create it before creating issues.

## Creating an Issue

1. Determine the next ID: scan `.plank/issues/` for the highest number with the project prefix, increment by 1.
2. Create `{ID}.md` using this template:

```markdown
---
id: {PREFIX}-{NNN}
title: {imperative verb phrase}
status: todo
priority: {critical|high|medium|low}
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
---

# {title}

## Description

{What needs to be done and why.}

## Acceptance Criteria

- [ ] {observable, verifiable behavior}
- [ ] {another criterion}
```

### Naming Rules

- `PREFIX`: 2–6 uppercase letters (project or category identifier)
- `NUMBER`: zero-padded to 3+ digits (`001`, `042`, `100`)
- Filename MUST match `id` field exactly
- Never rename files after creation

## Status Lifecycle

```plaintext
icebox → todo → in-progress → review → done
                     ↓                    ↑
                  blocked ────────────────┘
```

| Status        | Meaning                            |
|---------------|------------------------------------|
| `icebox`      | Idea captured, not yet prioritised |
| `todo`        | Prioritised, ready to pick up      |
| `in-progress` | Actively being worked on           |
| `review`      | Awaiting peer/QA review            |
| `done`        | Accepted and complete              |
| `blocked`     | Cannot proceed; set `blocked-by`   |
| `cancelled`   | Will not be done; reason noted     |

`done` and `cancelled` are **terminal**. Never reopen — create a new ticket and reference the original.

## Updating a Ticket

- Set `updated` to today's date on any meaningful change.
- Use frontmatter `blocked-by: [ID-001]` when a ticket cannot proceed.
- Keep `blocked-by`/`blocks` symmetric if using both directions.
- All frontmatter keys use **kebab-case**.

## Epics

- Epic files live in `.plank/epics/`, named `{EPIC-ID}-{slug}.md`.
- Issues reference an epic via `epic: EP-01-auth` in frontmatter.
- Epic format is the project's choice.

## Conventions

### Commit messages

```plaintext
plank({ID}): {action} [{new-status}]
```

Examples: `plank(PROJ-042): start work [in-progress]`, `plank(PROJ-042): close as done [done]`
