---
name: plank-black
description: >-
  Enforce PLANK BLACK project management in AI-assisted workflows. Use when
  starting or ending a work session, creating or closing tickets, running GC,
  or managing any project that follows the PLANK BLACK standard. This skill
  handles STATUS.md sync, Action Log, Stopped-at tracking, and session protocol.
---

# PLANK BLACK — AI Agent Workflow

Canonical spec: [PLANK_BLACK.md](../../PLANK_BLACK.md)

There is exactly one correct way. Follow it.

---

## 1. Session Start

Every session begins here. No exceptions.

1. Read `.plank/STATUS.md`.
2. Identify the target ticket from the **Active** table.
3. Read that ticket's latest Action Log entry and `**Stopped at**` line.
4. Resume from the exact point described.

If `.plank/STATUS.md` does not exist, create it using the format in [§ STATUS.md](#statusmd-format).

---

## 2. Creating a Ticket

Scan `.plank/issues/` for the highest number with the project prefix, increment by 1. Create `{ID}.md`:

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

## Action Log

### {YYYY-MM-DD}
- Created ticket
```

### Body Section Order (mandatory)

```plaintext
## Description              ← REQUIRED
## Acceptance Criteria      ← REQUIRED (may be empty only in icebox/todo)
## Action Log               ← REQUIRED
## Implementation Notes     ← OPTIONAL
## Completion Note          ← REQUIRED when deliverable diverged from Description
## Notes                    ← OPTIONAL
```

Sections MUST appear in this order. Optional sections may be omitted but MUST NOT appear out of order.

---

## 3. Status Lifecycle

```plaintext
icebox ──> todo ──> in-progress ──┬── review ──> done
                        │         │               ▲
                        ▼         └───────────────┘
                     blocked
                        │
                        ▼
                    cancelled
```

### Allowed Transitions

| From          | To                                       |
|---------------|------------------------------------------|
| `icebox`      | `todo`, `cancelled`                      |
| `todo`        | `in-progress`, `cancelled`               |
| `in-progress` | `done`, `review`, `blocked`, `cancelled` |
| `review`      | `done`, `in-progress`, `cancelled`       |
| `blocked`     | `in-progress`, `cancelled`               |

All other transitions are **forbidden**.

### Preconditions for `done`

Before marking any ticket `done`, verify ALL of:

- [ ] Every `- [ ]` in Acceptance Criteria is `- [x]`
- [ ] If deliverable diverged from Description → Completion Note exists
- [ ] `updated` field is set to today's date

### Preconditions for other transitions

- → `blocked`: `blocked-by` field contains at least one valid issue ID
- → `cancelled`: Reason is stated in the Action Log

`done` and `cancelled` are **terminal**. Never reopen. Use [Regression Protocol](#9-regression-protocol) instead.

---

## 4. Frontmatter Rules

### Required Fields

`id`, `title`, `status`, `priority`, `created`, `updated` — all mandatory on every ticket.

### `updated` Rule

Set `updated` to today's date whenever ANY of:

- A frontmatter field changes
- An Acceptance Criteria checkbox changes
- The Action Log receives a new entry

### `blocked-by` — Canonical Direction

`blocked-by` is the source of truth. `blocks` is derived and optional.

```yaml
blocked-by: [PROJ-039]
```

When the blocker is resolved, clear `blocked-by` on the unblocked ticket.

### `related` — General Cross-References

```yaml
related: [PROJ-011, PROJ-045]
```

Use for regressions, follow-up work, or any ticket sharing context with another.

---

## 5. Action Log

The Action Log is the primary tool for session continuity. Entries are in **reverse chronological order** (newest first).

### Format

```markdown
## Action Log

### YYYY-MM-DD
- What was done, decided, or discovered
- **Stopped at**: {exact resumption point}
```

### Tags

| Tag          | When to use                                          |
|--------------|------------------------------------------------------|
| `[DECISION]` | Fork in the road — record chosen, rejected, and why  |
| `[SPAWN]`    | New ticket discovered — include the new ticket ID    |
| `[SCOPE]`    | Acceptance Criteria or Description changed           |
| `[API]`      | A public API contract was added, changed, or removed |

Tags are optional on routine entries.

### `[DECISION]` Format

```markdown
- [DECISION] {topic}
  - Chose {option A}: {reason}
  - Rejected {option B}: {reason}
```

A decision MUST record at least one rejected alternative.

### `**Stopped at**` Rule

When a ticket remains in a **non-terminal** status at session end, the last Action Log entry MUST end with `**Stopped at**:`. It MUST be specific enough for someone with no context to find where to continue.

Tickets reaching `done` or `cancelled` do not need `**Stopped at**`.

**Good**: `**Stopped at**: unit test for rate limiter, tests/test_rate_limit.py:45, failing on window reset edge case`

**Bad**: `**Stopped at**: rate limiter tests`

---

## 6. Acceptance Criteria

- Every criterion MUST describe **observable, verifiable behavior**.
- When status leaves `todo`, at least one criterion MUST exist.
- When status enters `done`, every `- [ ]` MUST be `- [x]`.
- Criteria MUST NOT be deleted. If invalid, strike through and log with `[SCOPE]` tag.

---

## 7. Completion Note

A Completion Note MUST be added when the final deliverable **diverges** from the original Description. When all ACs are checked and the deliverable matches, it MAY be omitted.

```markdown
## Completion Note

YYYY-MM-DD

{What was delivered and how it differs from the original Description.}

Deliverables:
- {file path, endpoint, or measurable output}
```

---

## 8. Problem-Solving Protocol

**MUST** follow for debugging, diagnosis, or non-obvious technical decisions. **RECOMMENDED** for straightforward implementation.

```plaintext
OBSERVE ──> DIAGNOSE ──> VALIDATE ──> ACT ──> REFLECT
   ▲                                            │
   └────────────────────────────────────────────┘
```

- **OBSERVE**: Collect facts only. No interpretation. No code.
- **DIAGNOSE**: Propose **at least two** hypotheses. One hypothesis is forbidden (anchoring bias).
- **VALIDATE**: Define falsifiable test per hypothesis. Run them. Record results.
- **ACT**: Implement validated hypothesis. Code only after validation.
- **REFLECT**: Did the result match? If not, return to OBSERVE.

Hard rules:

- No code during OBSERVE or DIAGNOSE.
- Same approach fails twice → MUST produce new hypothesis. Third attempt forbidden.
- Log steps: `[OBSERVE]`, `[DIAGNOSE]`, `[VALIDATE]`, `[ACT]`, `[REFLECT]`.

---

## 9. Regression Protocol

A completed ticket's behavior has broken.

1. **Do not reopen** the original ticket. Terminal states are terminal.
2. Create a new ticket with `related: [{ORIGINAL-ID}]` in frontmatter.
3. In the Description, state what regressed and reference the original.
4. The original ticket remains intact and unmodified.

---

## 10. Spawn Protocol

Discovered work that belongs in a separate ticket while working on ticket A:

1. **Immediately** create a new ticket file (`todo` or `icebox`).
2. In ticket A's Action Log: `[SPAWN] {NEW-ID} — {one-line reason}`.
3. If the new ticket blocks A: add to A's `blocked-by`, change A to `blocked`.
4. Do **not** do the spawned work inside ticket A (unless trivial, < 5 minutes).

---

## 11. Session End

At the end of every work session, perform ALL steps in order:

1. **Action Log**: Add entry to every ticket touched. For non-terminal tickets, end with `**Stopped at**:`.
2. **Frontmatter**: Set `updated` to today on every ticket touched.
3. **STATUS.md**: Sync the Active table (see [format below](#statusmd-format)).
4. **Epic sync**: If any ticket's status changed, update its parent epic's issue table.
5. **Feedback**: Ask — "Any API gotchas, debugging insights, or architectural learnings worth capturing?" If yes, write a feedback file to `.plank/feedbacks/`.
6. **GC check**: If `Last reviewed` in STATUS.md is older than 7 days, run the [GC Ritual](#13-gc-ritual).

### Sprint Close

At the end of an intensive work period (regardless of 7-day timer):

1. Sync all epic issue tables.
2. Verify `blocked-by` references point to existing, non-terminal tickets.
3. Write missing Completion Notes for divergent deliverables.
4. Capture a retrospective feedback file if the sprint touched 5+ tickets.

---

## 12. STATUS.md Format

```markdown
# Project Status

Last reviewed: YYYY-MM-DD

## Active

| ID       | Title             | Status      | Stopped at                              |
|----------|-------------------|-------------|-----------------------------------------|
| PROJ-012 | Add rate limiting | in-progress | unit test `tests/test_rate_limit.py:45` |
| PROJ-015 | Fix auth redirect | blocked     | waiting on PROJ-012                     |

## Recently Done

| ID       | Title        | Completed  |
|----------|--------------|------------|
| PROJ-011 | DB migration | 2026-03-13 |
```

Rules:

- **Active**: all `todo`, `in-progress`, `review`, `blocked` tickets. `Stopped at` mirrors the ticket's latest.
- **Recently Done**: tickets completed since last GC.
- Update at end of every session.

---

## 13. GC Ritual

Run at least every 7 days, or at Sprint Close.

Perform in order:

1. **STATUS.md sync** — Active table matches actual statuses. Move `done`/`cancelled` to Recently Done.
2. **Action Log freshness** — Every `in-progress` ticket has an entry within 7 days. If not → `blocked` or `icebox`.
3. **Blocked review** — Verify each blocker is still valid. If resolved → `in-progress`, clear `blocked-by`.
4. **Reference check** — All `blocked-by` references point to existing tickets. Remove references to `done`/`cancelled`.
5. **Epic sync** — Every epic's issue table matches child statuses.
6. **Stale icebox** — `icebox` tickets older than 30 days with no Action Log → promote or cancel.
7. **Update `Last reviewed`** — Set to today.
8. **Clear old Recently Done** — Remove entries older than 30 days.

---

## 14. Epic Rules

### Issue Table Format

```markdown
## Issues

| ID       | Title             | Status      |
|----------|-------------------|-------------|
| PROJ-001 | Implement auth    | done        |
| PROJ-002 | Add rate limiting | in-progress |
```

### Epic Body Sections

```plaintext
## Overview     ← REQUIRED
## Issues       ← REQUIRED (table format)
## Notes        ← OPTIONAL
```

### Status Derivation

Epic status is derived from children:

| Children                                  | Epic status   |
|-------------------------------------------|---------------|
| All `icebox`                              | `icebox`      |
| Any `todo`, none `in-progress`+           | `todo`        |
| Any `in-progress`, `review`, or `blocked` | `in-progress` |
| All `done` or `cancelled`                 | `done`        |

---

## 15. Feedback Files

Location: `.plank/feedbacks/` (or project-level `feedbacks/`)

```markdown
---
id: FB-{NNN}
title: {descriptive title}
created: YYYY-MM-DD
---

# {title}

{API gotchas, debugging insights, retrospectives, architectural learnings.}
```

Write one after discovering non-obvious API behavior, after a productive debugging session, or at Sprint Close.
