# PLANK BLACK

**The Opinionated Standard — One Way, the Best Way**
Version: 1.1.0 | License: CC0

> PLANK BLACK is a **strict superset** of [PLANK](./PLANK.md).
> Every PLANK BLACK-compliant project automatically conforms to PLANK.
> Where PLANK leaves room for interpretation, PLANK BLACK removes it.

---

## Table of Contents

1. [Principles](#1-principles)
2. [Status Lifecycle](#2-status-lifecycle)
3. [Frontmatter Discipline](#3-frontmatter-discipline)
4. [Body Sections](#4-body-sections)
5. [Acceptance Criteria](#5-acceptance-criteria)
6. [Action Log](#6-action-log)
7. [Completion Note](#7-completion-note)
8. [Problem-Solving Protocol](#8-problem-solving-protocol)
9. [STATUS.md — The Entry Point](#9-statusmd--the-entry-point)
10. [Epic Rules](#10-epic-rules)
11. [Session Protocol](#11-session-protocol)
12. [Spawn Protocol](#12-spawn-protocol)
13. [Regression Protocol](#13-regression-protocol)
14. [GC Ritual](#14-gc-ritual)
15. [Feedback Files](#15-feedback-files)
16. [Conformance Checklist](#16-conformance-checklist)
17. [Changelog](#17-changelog)

---

## 1. Principles

PLANK says: *"Define the minimum shared contract."*

PLANK BLACK says: **There is exactly one correct way. Follow it.**

- Every rule uses **MUST** or **MUST NOT**. There is no "recommended" or "consider".
- If a rule is not listed here, PLANK's base spec applies.
- Ambiguity is a bug. If you are unsure how to do something, this document is incomplete — file an issue.

---

## 2. Status Lifecycle

### State Machine

```plaintext
icebox ──> todo ──> in-progress ──┬── review ──> done
                        │         │               ▲
                        ▼         └───────────────┘
                     blocked
                        │
                        ▼
                    cancelled

Any non-terminal state ──> cancelled
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

### Terminal States

`done` and `cancelled` are terminal. A ticket in a terminal state MUST NOT be reopened. See [Regression Protocol](#13-regression-protocol) for handling regressions.

### Transition Preconditions

| Transition        | Precondition                                                         |
|-------------------|----------------------------------------------------------------------|
| any → `done`      | All Acceptance Criteria checkboxes are `[x]`                         |
| any → `done`      | [Completion Note](#7-completion-note) exists if deliverable diverged |
| any → `done`      | `updated` field is set to today's date                               |
| any → `blocked`   | `blocked-by` field contains at least one valid issue ID              |
| any → `cancelled` | Reason is stated in the Action Log                                   |

---

## 3. Frontmatter Discipline

### Required Fields (same as PLANK)

| Field      | Type     | Constraint                                    |
|------------|----------|-----------------------------------------------|
| `id`       | `string` | `{PREFIX}-{NNN}`, matches filename            |
| `title`    | `string` | Imperative verb phrase                        |
| `status`   | `enum`   | One of the seven defined values               |
| `priority` | `enum`   | `critical` \| `high` \| `medium` \| `low`     |
| `created`  | `date`   | ISO 8601, set once at creation, never changed |
| `updated`  | `date`   | ISO 8601, see update rule below               |

### `updated` Rule

The `updated` field MUST be set to the current date whenever **any** of the following occurs:

- A frontmatter field changes
- An Acceptance Criteria checkbox is checked or unchecked
- The Action Log receives a new entry

No exceptions.

### `blocked-by` — Canonical Direction

`blocked-by` is the **source of truth** for blocking relationships:

- A ticket's `blocked-by` field lists the IDs that prevent it from proceeding.
- `blocks` is **derived** — it can be computed by scanning all tickets' `blocked-by` fields. Maintaining `blocks` manually is permitted but not required.
- When the blocking relationship is resolved, clear `blocked-by` on the unblocked ticket.

### `related` — General Cross-References

The `related` frontmatter field links tickets that are connected but do not have a blocking relationship:

```yaml
related: [PROJ-011, PROJ-045]
```

Use `related` for regressions (see [§13](#13-regression-protocol)), follow-up work, or any ticket that shares context with another.

---

## 4. Body Sections

Issue body sections MUST appear in the following order. Sections marked **required** MUST be present. Optional sections MAY be omitted entirely but MUST NOT appear out of order.

```markdown
# {title}

## Description              ← REQUIRED

## Acceptance Criteria      ← REQUIRED (may be empty only in icebox/todo)

## Action Log               ← REQUIRED (first entry added when work begins)

## Implementation Notes     ← OPTIONAL

## Completion Note          ← REQUIRED when deliverable diverged from Description

## Notes                    ← OPTIONAL
```

### Epic Body Sections

```markdown
# {title}

## Overview                 ← REQUIRED

## Issues                   ← REQUIRED (table format, see §10)

## Notes                    ← OPTIONAL
```

---

## 5. Acceptance Criteria

### Content Rules

- Every criterion MUST describe **observable, verifiable behavior**.
- A reviewer who has never read the code MUST be able to verify each criterion by running or testing the system.

**Good:**

```markdown
- [ ] User receives a 401 response when the token is expired
- [ ] Export file contains all records matching the date filter
```

**Bad:**

```markdown
- [ ] AuthService.validate() raises TokenExpiredError
- [ ] Implement proper error handling
```

### State Rules

- When status leaves `todo`, at least one criterion MUST exist.
- When status enters `done`, every `- [ ]` MUST be `- [x]`.
- Criteria MUST NOT be deleted. If a criterion becomes invalid, strike it through and log the reason in the Action Log with a `[SCOPE]` tag.

---

## 6. Action Log

The Action Log is the chronological record of work on a ticket. It is the primary tool for session continuity.

### Format

```markdown
## Action Log

### YYYY-MM-DD
- What was done, decided, or discovered
- **Stopped at**: {exact point where work was interrupted}
```

Entries are in **reverse chronological order** (newest first).

### Tags

Use tags at the start of a log entry to classify it:

| Tag          | Meaning                                                                  |
|--------------|--------------------------------------------------------------------------|
| `[DECISION]` | A fork in the road — records what was chosen, what was rejected, and why |
| `[SPAWN]`    | New ticket discovered during work — includes the new ticket ID           |
| `[SCOPE]`    | Acceptance Criteria or Description changed — records what and why        |
| `[API]`      | A public API contract was added, changed, or removed                     |

Tags are not required on every entry. Plain entries (no tag) record routine progress.

### `**Stopped at**` Rule

When a ticket remains in a **non-terminal** status at session end, the last Action Log entry MUST end with a `**Stopped at**:` line. This is the **exact resumption point** for the next session. It MUST be specific enough that someone with no prior context can find where to continue.

Tickets that reach `done` or `cancelled` during the session do not need a `**Stopped at**` line — there is nothing to resume.

**Good:**

```markdown
- **Stopped at**: unit test for rate limiter, `tests/test_rate_limit.py:45`, failing on edge case where window resets mid-request
```

**Bad:**

```markdown
- **Stopped at**: rate limiter tests
```

### `[DECISION]` Format

```markdown
- [DECISION] {topic}
  - Chose {option A}: {reason}
  - Rejected {option B}: {reason}
```

A decision MUST record at least one rejected alternative. A decision with no alternatives considered is not a decision — it is an assumption.

---

## 7. Completion Note

A Completion Note MUST be added when the final deliverable **diverges** from the original Description. When all Acceptance Criteria are checked and the deliverable matches the Description, the Completion Note MAY be omitted — the Action Log and ACs are sufficient.

### Format

```markdown
## Completion Note

YYYY-MM-DD

{One sentence: what was delivered and how it differs from the original Description.}

Deliverables:
- {file path, endpoint, measurable output, or artifact}
```

### Rules

- The date MUST match the `updated` field.
- The divergence from the original Description MUST be stated explicitly.
- Deliverables MUST be concrete — file paths, URLs, or measurable outputs. Not "improved performance" but "p95 latency reduced from 800ms to 200ms".

---

## 8. Problem-Solving Protocol

When a ticket involves **diagnosing an issue, debugging, or making a non-obvious technical decision**, this protocol MUST be followed. For straightforward implementation tickets, it is RECOMMENDED but not required. Log each step in the Action Log.

### The Five Steps

```plaintext
OBSERVE ──> DIAGNOSE ──> VALIDATE ──> ACT ──> REFLECT
   ▲                                            │
   └────────────────────────────────────────────┘
```

**OBSERVE** — Collect facts. Record only what is directly observable. No interpretation.

**DIAGNOSE** — Propose **at least two** possible causes or approaches. Writing only one hypothesis is forbidden — it leads to anchoring bias.

**VALIDATE** — For each hypothesis, define a **falsifiable test**. Run the tests. Record results.

**ACT** — Implement the validated hypothesis. Only write code after validation.

**REFLECT** — Did the result match the expectation? If not, return to OBSERVE with new data.

### Hard Rules

- Writing code during OBSERVE or DIAGNOSE is **forbidden**.
- If the same approach fails twice, you MUST return to DIAGNOSE and produce a new hypothesis. Repeating a failed approach a third time is **forbidden**.
- Log the current step in the Action Log: `[OBSERVE]`, `[DIAGNOSE]`, `[VALIDATE]`, `[ACT]`, `[REFLECT]`.

### Example

```markdown
### 2026-03-15
- [OBSERVE] API returns 500 on /users endpoint. Logs show: "connection pool exhausted after 30s"
- [DIAGNOSE] Hypothesis A: pool size too small for current load / Hypothesis B: N+1 query holding connections open
- [VALIDATE] Ran EXPLAIN ANALYZE on the query — 847 sequential scans confirmed. Pool size is 20, sufficient for normal load.
- [ACT] Added eager loading to user->roles relation. Response time dropped to 180ms.
- [REFLECT] Confirmed fix. Pool exhaustion was a symptom, not the cause.
```

---

## 9. STATUS.md — The Entry Point

Every PLANK BLACK project MUST have a `.plank/STATUS.md` file. This is the **single entry point** when starting work.

### Format

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

### Rules

- **Active** table lists all tickets with status `todo`, `in-progress`, `review`, or `blocked`. The `Stopped at` column mirrors the latest `**Stopped at**` from the ticket's Action Log.
- **Recently Done** table lists tickets completed since the last GC. Cleared during GC.
- `Last reviewed` is updated during GC (see [§14](#14-gc-ritual)).
- STATUS.md MUST be updated at the end of every work session (see [§11](#11-session-protocol)).

---

## 10. Epic Rules

### Issue Table Format

The `## Issues` section in an epic MUST use this exact table format:

```markdown
## Issues

| ID       | Title             | Status      |
|----------|-------------------|-------------|
| PROJ-001 | Implement auth    | done        |
| PROJ-002 | Add rate limiting | in-progress |
```

### Sync Rule

When an issue's status changes, the epic's issue table MUST be updated in the same session.

### Epic Status Derivation

An epic's status is **derived** from its children:

| Child states                              | Epic status   |
|-------------------------------------------|---------------|
| All `icebox`                              | `icebox`      |
| Any `todo`, none `in-progress`+           | `todo`        |
| Any `in-progress`, `review`, or `blocked` | `in-progress` |
| All `done` or `cancelled`                 | `done`        |

An epic MUST NOT have a status that contradicts this table.

---

## 11. Session Protocol

### Starting a Session

1. Open `.plank/STATUS.md`.
2. Identify the target ticket from the **Active** table.
3. Read that ticket's latest Action Log entry and `**Stopped at**` line.
4. Begin work.

### Ending a Session

1. Add an Action Log entry to every ticket touched. For tickets remaining in non-terminal status, end with `**Stopped at**:`.
2. Update `updated` in frontmatter of every ticket touched.
3. Update `.plank/STATUS.md` — sync the Active table.
4. Capture learnings: any API gotchas, debugging insights, or architectural decisions worth preserving SHOULD be written to a feedback file (see [§15](#15-feedback-files)).
5. If the last GC was more than 7 days ago, run the [GC Ritual](#14-gc-ritual).

### Sprint Close

At the end of an intensive work period (multiple sessions or rapid iteration), run a lightweight wrap-up **regardless of the 7-day GC timer**:

1. Sync epic issue tables with current ticket statuses.
2. Verify `blocked-by` references point to existing, non-terminal tickets.
3. Write any missing Completion Notes for tickets where the deliverable diverged.
4. Capture a sprint retrospective feedback file if the sprint touched 5+ tickets.

---

## 12. Spawn Protocol

When working on ticket A, you discover work that belongs in a separate ticket.

### Steps

1. **Immediately** create a new ticket file with status `todo` or `icebox`.
2. In ticket A's Action Log, add: `[SPAWN] {NEW-ID} — {one-line reason}`.
3. If the new ticket blocks A, add the new ticket's ID to A's `blocked-by` and change A's status to `blocked`.
4. Do **not** do the spawned work inside ticket A, unless it is a trivial fix completable in under 5 minutes.

---

## 13. Regression Protocol

A completed ticket's behavior has broken.

### Steps

1. **Do not reopen** the original ticket. Terminal states are terminal.
2. Create a new ticket. In its frontmatter, add the original ticket to `related` (see [§3](#3-frontmatter-discipline)).
3. In the new ticket's Description, state what behavior regressed and reference the original ticket.
4. The original ticket's history remains intact and unmodified.

---

## 14. GC Ritual

GC (Garbage Collection) is a periodic maintenance ritual. Run it at least once every 7 days, or when triggered by the Session Protocol.

### Checklist

Perform every item in order:

1. **STATUS.md sync** — Verify every ticket in the Active table matches its actual status. Remove `done`/`cancelled` tickets (move to Recently Done). Add newly active tickets.
2. **Action Log freshness** — Every `in-progress` ticket MUST have an Action Log entry within the last 7 days. If not, move the ticket to `blocked` or `icebox` with an explanation.
3. **Blocked ticket review** — For every `blocked` ticket, verify the blocker is still valid. If resolved, update status to `in-progress` and clear `blocked-by`.
4. **Reference check** — Verify all `blocked-by` references point to existing tickets. Remove references to tickets that are now `done` or `cancelled`.
5. **Epic sync** — Every epic's issue table matches the actual status of its child tickets.
6. **Stale icebox** — Any `icebox` ticket older than 30 days with no Action Log entry: promote to `todo` or `cancel`.
7. **Update `Last reviewed`** — Set `Last reviewed` in STATUS.md to today's date.
8. **Clear Recently Done** — Remove entries older than 30 days from the Recently Done table.

---

## 15. Feedback Files

Feedback files capture API gotchas, debugging insights, architectural learnings, and process retrospectives that would otherwise be lost between sessions.

### Location

Feedback files live in `.plank/feedbacks/` (or a project-level `feedbacks/` directory).

### Format

```markdown
---
id: FB-{NNN}
title: {descriptive title}
created: YYYY-MM-DD
---

# {title}

{Free-form content: gotchas, learnings, retrospectives.}
```

### When to Write

- After discovering a non-obvious API behavior or undocumented constraint.
- After a debugging session that produced reusable insight.
- At the end of an intensive sprint (see [Sprint Close](#sprint-close) in §11).

Feedback files are the highest-ROI artifact for future developers. When in doubt, write one.

---

## 16. Conformance Checklist

A project conforms to PLANK BLACK if it satisfies all PLANK conformance requirements **plus** the following:

- [ ] `.plank/STATUS.md` exists and has been updated within the last 7 days
- [ ] Every issue body follows the section order defined in [§4](#4-body-sections)
- [ ] Every `in-progress` or `review` ticket has an Action Log with a `**Stopped at**` line in its latest entry
- [ ] Every `done` ticket has all Acceptance Criteria checked `[x]`
- [ ] Every `done` ticket where the deliverable diverged from the Description has a Completion Note
- [ ] Every `cancelled` ticket has a reason in its Action Log
- [ ] Every `blocked` ticket has a non-empty `blocked-by` field
- [ ] All `blocked-by` references point to existing tickets
- [ ] No forbidden status transitions exist in the ticket history
- [ ] Epic issue tables are in sync with child ticket statuses
- [ ] GC has been performed within the last 7 days

---

## 17. Changelog

| Version | Date       | Notes                                                                                                                                                                                                                      |
|---------|------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1.1.0   | 2026-03-23 | Feedback-driven revision: `blocked-by` canonical, Completion Note conditional, `in-progress → done` transition, `[API]` tag, `related` field, Problem-Solving Protocol scoped, Sprint Close ritual, Feedback Files section |
| 1.0.0   | 2026-03-15 | Initial release                                                                                                                                                                                                            |

---

*PLANK BLACK is released under CC0. Copy, fork, adapt freely.*
*Canonical spec: `https://raw.githubusercontent.com/allen2c/PLANK/main/PLANK_BLACK.md`*
*GitHub Pages: `https://allen2c.github.io/PLANK/PLANK_BLACK`*
