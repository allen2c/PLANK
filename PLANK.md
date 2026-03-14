# PLANK Standard

**Plaintext Local Agile Notes & Kanban**
Version: 1.2.0 | License: CC0

> A file-system-native project management standard for Git repositories.
> Any repo that follows this spec can be read, written, and automated by humans, CLI tools, and AI agents without external services.

---

## Table of Contents

1. [Zen of PLANK](#1-zen-of-plank)
2. [Philosophy](#2-philosophy)
3. [Directory Layout](#3-directory-layout)
4. [Issue File Format](#4-issue-file-format)
5. [Frontmatter Schema](#5-frontmatter-schema)
6. [Status Lifecycle](#6-status-lifecycle)
7. [Naming Conventions](#7-naming-conventions)
8. [Epics](#8-epics)
9. [Linking & Relations](#9-linking--relations)
10. [Conventions](#10-conventions)
11. [Consuming This Standard](#11-consuming-this-standard)
12. [PLANK BLACK](#12-plank-black)
13. [Changelog](#13-changelog)

---

## 1. Zen of PLANK

- Files are the database. Git is the audit log.
- Define the minimum shared contract. Everything else is each project's choice.
- A convention that fits in one sentence is better than a configuration file.
- If a tool can't read it with `grep` and a YAML parser, it's too complex.
- Start with issues. Add structure only when the pain arrives.

---

## 2. Philosophy

- **Files are the database.** No external service, no proprietary format.
- **Git is the audit log.** Every state change is a commit.
- **YAML Frontmatter is the schema.** Structured metadata lives at the top; human narrative lives below.
- **AI-readable by design.** Any agent that can `fetch` a URL or read a file can participate in task management.

---

## 3. Directory Layout

```plaintext
<repo-root>/
└── .plank/
    ├── issues/             # flat issue store — primary source of truth
    │   ├── PROJ-001.md
    │   ├── PROJ-002.md
    │   └── PROJ-003.md
    └── epics/              # epic definitions (optional)
        └── EP-01-auth.md
```

`.plank/` is a hidden directory by convention so it does not clutter the repo root, but is always present and discoverable by tooling. Issue status is tracked exclusively via the `status` frontmatter field — there are no subfolders for status.

Only `issues/` is required. Projects may add directories as needed.

---

## 4. Issue File Format

Every issue is a single `.md` file. The structure is:

```plaintext
[YAML Frontmatter block]
[blank line]
# {title}
[blank line]
## Description
{free-form markdown}
[blank line]
## Acceptance Criteria
- [ ] criterion 1
- [ ] criterion 2
[blank line]
## Notes / Comments
### {ISO-date} @{author}
{comment body}
```

### Full Example

```markdown
---
id: PROJ-042
title: Implement dark mode toggle
status: in-progress
priority: high
assignee: yu-jung
labels: [frontend, ui, accessibility]
epic: EP-01-auth
created: 2026-02-26
updated: 2026-02-28
blocked-by: [PROJ-039]
---

# Implement dark mode toggle

## Description
Add a toggle button in the top navigation bar that switches between light and dark themes.
Should respect `prefers-color-scheme` on first load.

## Acceptance Criteria
- [ ] Toggle button renders in nav bar
- [ ] State persists via localStorage
- [ ] Respects OS-level `prefers-color-scheme` on initial render
- [ ] All existing components pass contrast ratio AA

## Notes / Comments

### 2026-02-26 @yu-jung
Initial spec drafted. Waiting on design token decisions.

### 2026-02-28 @wei-chen
Design tokens confirmed. Unblocking this after PROJ-039 merges.
```

---

## 5. Frontmatter Schema

All fields use YAML. Only the following fields are **required**:

| Field      | Type     | Description                                                        |
|------------|----------|--------------------------------------------------------------------|
| `id`       | `string` | Unique issue identifier. Format: `{PREFIX}-{NNN}`. e.g. `PROJ-042` |
| `title`    | `string` | Short imperative sentence describing the task                      |
| `status`   | `enum`   | See [Status Lifecycle](#6-status-lifecycle)                        |
| `priority` | `enum`   | `critical` \| `high` \| `medium` \| `low`                          |
| `created`  | `date`   | ISO 8601 date: `YYYY-MM-DD`                                        |
| `updated`  | `date`   | ISO 8601 date of last meaningful change                            |

Projects may add any additional fields (e.g. `assignee`, `labels`, `estimate`, `due`, `epic`, `blocked-by`). Frontmatter keys should use **kebab-case** to ensure consistency across tools and projects.

---

## 6. Status Lifecycle

```plaintext
icebox --> todo --> in-progress --> review --> done
                       |                       ^
                       +------> blocked -------+
                                   |
                              cancelled
```

| Status        | Meaning                                       |
|---------------|-----------------------------------------------|
| `icebox`      | Idea captured, not yet prioritised            |
| `todo`        | Prioritised, ready to be picked up            |
| `in-progress` | Actively being worked on                      |
| `review`      | PR open or awaiting peer/QA review            |
| `done`        | Accepted and complete                         |
| `blocked`     | Cannot proceed; `blocked-by` should be set    |
| `cancelled`   | Will not be done; reason noted in Description |

`done` and `cancelled` are **terminal states**. Once a ticket reaches either state, it should not be reopened — create a new ticket and reference the original instead.

---

## 7. Naming Conventions

### Issue files

```plaintext
{ID}.md
```

Examples: `PROJ-001.md`, `API-042.md`, `BUG-007.md`

The **ID** follows the pattern `{PREFIX}-{NUMBER}`:

- `PREFIX`: 2-6 uppercase letters identifying the project or category
- `NUMBER`: zero-padded to 3 digits minimum (`001`, `042`, `100`)
- New issue number = highest existing number with the same prefix + 1

> Do not rename files after creation. The filename is the stable external reference.

---

## 8. Epics

An epic groups related issues into a deliverable phase — a coherent chunk of work that can be planned, completed, and reviewed together before moving on to the next phase.

Issues reference an epic via the `epic` frontmatter field:

```yaml
epic: EP-01-auth
```

Epic files live in `.plank/epics/` and follow the naming pattern `{EPIC-ID}-{slug}.md` (e.g. `EP-01-auth.md`). The format and content of epic files is not prescribed — projects decide what works for them.

Epics are optional. Use them when you need a level of structure above individual issues.

---

## 9. Linking & Relations

Reference issues by ID in prose or frontmatter:

```markdown
This task is blocked by PROJ-039 and will unblock PROJ-045.
```

In frontmatter arrays, use bare IDs:

```yaml
blocked-by: [PROJ-039]
blocks: [PROJ-045, PROJ-046]
```

When using both `blocked-by` and `blocks`, keep them **symmetric**: if A lists B in `blocked-by`, then B should list A in `blocks`, and vice versa.

---

## 10. Conventions

The following are recommended practices, not requirements. They reduce communication cost when adopted consistently.

### Commit messages

```plaintext
plank({ID}): {action} [{new-status}]
```

Examples:

```plaintext
plank(PROJ-042): create issue
plank(PROJ-042): start work [in-progress]
plank(PROJ-042): close as done [done]
```

This enables `git log --grep="plank(PROJ-042)"` to retrieve the full history of any issue.

### Comment format

Append comments under `## Notes / Comments` using:

```markdown
### {ISO-date} @{author}
{comment body}
```

This keeps async communication chronological and grep-friendly.

### Updating the `updated` field

Any frontmatter field change **must** update `updated` to the current date. Adding a comment alone does not require it.

---

## 11. Consuming This Standard

### For AI agents / automation

Any agent should begin a session by fetching this spec:

```plaintext
GET https://raw.githubusercontent.com/allen2c/PLANK/main/PLANK.md
```

If the project follows PLANK BLACK, also fetch:

```plaintext
GET https://raw.githubusercontent.com/allen2c/PLANK/main/PLANK_BLACK.md
```

The response is plain `text/markdown`. Parse frontmatter fields with any YAML library.

### Discovering PLANK in a repo

A PLANK-compliant repo **must** contain `.plank/issues/` at the root.

### Minimal conformance checklist

A repo is PLANK-compliant if it satisfies:

- [ ] `.plank/issues/` directory exists
- [ ] Every issue file has `id`, `title`, `status`, `priority`, `created`, `updated` in frontmatter
- [ ] `id` matches the filename (e.g. `PROJ-042.md` has `id: PROJ-042`)
- [ ] `status` is one of the seven defined values
- [ ] Frontmatter keys use kebab-case
- [ ] No issue file is renamed after creation

---

## 12. PLANK BLACK

PLANK defines the minimum shared contract. **PLANK BLACK** is the opinionated strict superset — there is only one way to do things, and that is the best way.

A project that follows PLANK BLACK automatically conforms to PLANK. PLANK BLACK adds mandatory workflows, stricter transition rules, and session protocols that eliminate ambiguity.

> **[PLANK_BLACK.md](./PLANK_BLACK.md)**

---

## 13. Changelog

| Version | Date       | Notes                                                             |
|---------|------------|-------------------------------------------------------------------|
| 1.2.0   | 2026-03-15 | Add terminal state rule; symmetric linking; introduce PLANK BLACK |
| 1.1.0   | 2026-03-06 | Add Zen; add Epics; trim optional fields; clarify conventions     |
| 1.0.0   | 2026-02-26 | Initial release                                                   |

---

*PLANK is released under CC0. Copy, fork, adapt freely.*
*Canonical spec: `https://raw.githubusercontent.com/allen2c/PLANK/main/PLANK.md`*
*GitHub Pages: `https://allen2c.github.io/PLANK/PLANK`*
