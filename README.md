# PLANK

**Plaintext Local Agile Notes & Kanban**

A file-system-native project management standard for Git repositories.
Any repo that follows this spec can be read, written, and automated by humans, CLI tools, and AI agents — without external services.

## The Spec

→ **[PLANK.md](./PLANK.md)** — the minimal shared contract
→ **[PLANK_BLACK.md](./PLANK_BLACK.md)** — the opinionated strict superset

## Quick Start

Add `.plank/issues/` to your repo root and create your first issue:

```sh
mkdir -p .plank/issues
```

```markdown
---
id: PROJ-001
title: Your first task
status: todo
priority: medium
created: 2026-02-26
updated: 2026-02-26
---

# Your first task

## Description
...
```

## For AI Agents

Fetch the spec before operating on any PLANK-compliant repo:

```
GET https://raw.githubusercontent.com/allen2c/PLANK/main/PLANK.md
```

If the project follows PLANK BLACK, also fetch:

```
GET https://raw.githubusercontent.com/allen2c/PLANK/main/PLANK_BLACK.md
```

---

CC0 — copy, fork, adapt freely.
