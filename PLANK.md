# PLANK Standard

**Plaintext Local Agile Notes & Kanban**
Version: 1.0.0 | License: CC0

> A file-system-native project management standard for Git repositories.
> Any repo that follows this spec can be read, written, and automated by humans, CLI tools, and AI agents without external services.

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [Directory Layout](#2-directory-layout)
3. [Issue File Format](#3-issue-file-format)
4. [Frontmatter Schema](#4-frontmatter-schema)
5. [Status Lifecycle](#5-status-lifecycle)
6. [Naming Conventions](#6-naming-conventions)
7. [Linking & Relations](#7-linking--relations)
8. [Git Conventions](#8-git-conventions)
9. [Sprint & Milestone Files](#9-sprint--milestone-files)
10. [Consuming This Standard](#10-consuming-this-standard)
11. [Minimal vs Full Layout](#11-minimal-vs-full-layout)
12. [Changelog](#12-changelog)

---

## 1. Philosophy

- **Files are the database.** No external service, no proprietary format.
- **Git is the audit log.** Every state change is a commit.
- **YAML Frontmatter is the schema.** Structured metadata lives at the top; human narrative lives below.
- **AI-readable by design.** Any agent that can `fetch` a URL or read a file can participate in task management.

---

## 2. Directory Layout

```plaintext
<repo-root>/
└── .plank/
    ├── issues/             # flat issue store — primary source of truth
    │   ├── PROJ-001.md
    │   ├── PROJ-002.md
    │   └── PROJ-003.md
    ├── sprints/            # sprint / milestone definitions (optional)
    │   └── 2026-S01.md
    └── epics/              # epic definitions (optional)
        └── EP-01-auth.md
```

`.plank/` is a hidden directory by convention so it does not clutter the repo root, but is always present and discoverable by tooling. Issue status is tracked exclusively via the `status` frontmatter field—there are no subfolders for status.

---

## 3. Issue File Format

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
sprint: 2026-S01
created: 2026-02-26
updated: 2026-02-28
estimate: 3
due: 2026-03-10
blocked-by: [PROJ-039]
blocks: []
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
Initial spec drafted. Waiting on design token decisions from EP-01.

### 2026-02-28 @wei-chen
Design tokens confirmed. Unblocking this after PROJ-039 merges.
```

---

## 4. Frontmatter Schema

All fields use YAML. Fields marked **required** must be present. All others are optional but recommended.

| Field        | Type                 | Required | Description                                                        |
|--------------|----------------------|----------|--------------------------------------------------------------------|
| `id`         | `string`             | ✅       | Unique issue identifier. Format: `{PREFIX}-{NNN}`. e.g. `PROJ-042` |
| `title`      | `string`             | ✅       | Short imperative sentence describing the task                      |
| `status`     | `enum`               | ✅       | See [Status Lifecycle](#5-status-lifecycle)                        |
| `priority`   | `enum`               | ✅       | `critical` \| `high` \| `medium` \| `low`                          |
| `created`    | `date`               | ✅       | ISO 8601 date: `YYYY-MM-DD`                                        |
| `updated`    | `date`               | ✅       | ISO 8601 date of last meaningful change                            |
| `assignee`   | `string`             | —        | GitHub username or identifier (no `@`)                             |
| `labels`     | `string[]`           | —        | Freeform tags. Lowercase, hyphen-separated                         |
| `epic`       | `string`             | —        | Epic file reference without extension. e.g. `EP-01-auth`           |
| `sprint`     | `string`             | —        | Sprint file reference. e.g. `2026-S01`                             |
| `estimate`   | `number`             | —        | Story points or hours (unit defined per project)                   |
| `due`        | `date`               | —        | ISO 8601 target completion date                                    |
| `blocked-by` | `string[]`           | —        | List of issue IDs this issue cannot proceed without                |
| `blocks`     | `string[]`           | —        | List of issue IDs this issue is blocking                           |
| `parent`     | `string`             | —        | Parent issue ID for sub-tasks                                      |
| `pr`         | `string \| string[]` | —        | Pull request URL(s) associated with this issue                     |

### Label Conventions (recommended, not enforced)

| Label              | Meaning                                |
|--------------------|----------------------------------------|
| `bug`              | Something is broken                    |
| `feat`             | New feature                            |
| `chore`            | Maintenance, deps, refactor            |
| `docs`             | Documentation only                     |
| `blocked`          | Cannot proceed (use with `blocked-by`) |
| `good-first-issue` | Suitable for newcomers                 |

---

## 5. Status Lifecycle

```plaintext
icebox ──► todo ──► in-progress ──► review ──► done
                        │                        ▲
                        └──────► blocked ────────┘
                                      │
                                 cancelled
```

| Status        | Meaning                                       |
|---------------|-----------------------------------------------|
| `icebox`      | Idea captured, not yet prioritised            |
| `todo`        | Prioritised, ready to be picked up            |
| `in-progress` | Actively being worked on                      |
| `review`      | PR open or awaiting peer/QA review            |
| `done`        | Accepted and complete                         |
| `blocked`     | Cannot proceed; `blocked-by` must be set      |
| `cancelled`   | Will not be done; reason noted in Description |

**Rule:** A transition from any status to `done` or `cancelled` requires updating the `updated` field.

---

## 6. Naming Conventions

### Issue files

```plaintext
{ID}.md
```

Examples: `PROJ-001.md`, `API-042.md`, `BUG-007.md`

The **ID** follows the pattern `{PREFIX}-{NUMBER}`:

- `PREFIX`: 2–6 uppercase letters identifying the project or category
- `NUMBER`: zero-padded to 3 digits minimum (`001`, `042`, `100`)

> Do not rename files after creation. The filename is the stable external reference.

### Sprint files

```plaintext
{YYYY}-S{NN}.md          # e.g. 2026-S01.md  (sequential)
{YYYY}-W{WW}.md          # e.g. 2026-W09.md  (ISO week)
```

### Epic files

```plaintext
{EPIC-ID}-{slug}.md      # e.g. EP-01-auth.md
```

---

## 7. Linking & Relations

### Intra-repo links

Reference issues by ID in prose or frontmatter:

```markdown
This task is blocked by PROJ-039 and will unblock PROJ-045.
```

In frontmatter arrays, use bare IDs:

```yaml
blocked-by: [PROJ-039]
blocks: [PROJ-045, PROJ-046]
```

For tools that support wiki-style links (Obsidian, Foam):

```markdown
See [[PROJ-039]] for context.
```

### Cross-repo links

Use full relative paths or URLs:

```yaml
blocked-by: ["https://github.com/org/other-repo/blob/main/.plank/issues/TASK-012.md"]
```

---

## 8. Git Conventions

### Commit message format

```plaintext
plank({ID}): {action} [{new-status}]
```

Examples:

```plaintext
plank(PROJ-042): create issue
plank(PROJ-042): start work [in-progress]
plank(PROJ-042): open for review [review]
plank(PROJ-042): close as done [done]
plank(PROJ-042): add comment
```

### Branch naming

```plaintext
plank/{ID}-{short-slug}
```

Example: `plank/PROJ-042-dark-mode-toggle`

### Pull Request title

When a PR resolves an issue, include the ID:

```plaintext
feat: dark mode toggle [PROJ-042]
```

### Auto-close keyword (GitHub)

```plaintext
closes .plank/issues/PROJ-042.md
```

> Standard GitHub `closes #N` syntax does not apply to file-based issues. Teams may automate status updates via GitHub Actions by parsing commit messages for the `plank({ID})` pattern.

---

## 9. Sprint & Milestone Files

Sprint files aggregate issue references and capture velocity targets.

```markdown
---
id: 2026-S01
title: Sprint 01 — Auth Foundation
start: 2026-02-24
end: 2026-03-07
goal: Ship user authentication end-to-end
capacity: 24
---

# Sprint 01 — Auth Foundation

## Goal
Ship user authentication end-to-end (registration, login, JWT refresh).

## Issues

| ID       | Title             | Assignee | Estimate | Status      |
|----------|-------------------|----------|----------|-------------|
| PROJ-039 | JWT middleware    | wei-chen | 3        | done        |
| PROJ-042 | Dark mode toggle  | yu-jung  | 3        | in-progress |
| PROJ-043 | Registration form | yu-jung  | 5        | todo        |

## Retrospective
*(filled at sprint close)*
```

---

## 10. Consuming This Standard

### For AI agents / automation

Any agent should begin a session by fetching this spec:

```plaintext
GET https://raw.githubusercontent.com/allen2c/PLANK/main/PLANK.md
```

The response is plain `text/markdown`. Parse frontmatter fields with any YAML library.

### Discovering PLANK in a repo

A PLANK-compliant repo **must** contain `.plank/` at the root.
A consuming tool **should** check for `.plank/issues/` and treat its contents as the issue store.

### Minimal conformance checklist

A repo is PLANK-compliant if it satisfies:

- [ ] `.plank/issues/` directory exists
- [ ] Every issue file has `id`, `title`, `status`, `priority`, `created`, `updated` in frontmatter
- [ ] `id` matches the filename (e.g. `PROJ-042.md` has `id: PROJ-042`)
- [ ] `status` is one of the seven defined values
- [ ] No issue file is renamed after creation

### Recommended tooling

| Tool                | Purpose                                      |
|---------------------|----------------------------------------------|
| `grep` / `ripgrep`  | Query issues by any frontmatter field        |
| `yq`                | Parse and mutate YAML frontmatter from shell |
| GitHub Actions      | Automate status transitions on PR events     |
| Obsidian + Dataview | Visual kanban and query dashboard            |
| Backlog.md          | CLI + web kanban for PLANK-style repos       |

### Example: list all `in-progress` issues (shell)

```sh
rg 'status: in-progress' .plank/issues/ -l
```

### Example: mark an issue done (yq)

```sh
yq -i '.status = "done" | .updated = "'$(date +%F)'"' .plank/issues/PROJ-042.md
```

---

## 11. Minimal vs Full Layout

| Feature                        | Minimal     | Full |
|--------------------------------|-------------|------|
| Issue files                    | ✅ required | ✅   |
| YAML Frontmatter (core fields) | ✅ required | ✅   |
| Sprint files                   | ❌ optional | ✅   |
| Epic files                     | ❌ optional | ✅   |
| Git commit convention          | ❌ optional | ✅   |
| Branch naming convention       | ❌ optional | ✅   |

Start minimal. Add layers as the team grows.

---

## 12. Changelog

| Version | Date       | Notes           |
|---------|------------|-----------------|
| 1.0.0   | 2026-02-26 | Initial release |

---

*PLANK is released under CC0. Copy, fork, adapt freely.*
*Canonical spec: `https://raw.githubusercontent.com/allen2c/PLANK/main/PLANK.md`*
