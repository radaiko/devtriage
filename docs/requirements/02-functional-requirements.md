# 02 — Functional Requirements

Requirement IDs are grouped by capability. Each requirement has a **priority**:
**MUST** (initial release), **SHOULD** (highly desired), **MAY** (future / nice).

Legend: 🔴 MUST · 🟠 SHOULD · 🟢 MAY

---

## Capture — `FR-CAP`

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-CAP-1 | 🔴 | The user can create a todo with at minimum a title. |
| FR-CAP-2 | 🔴 | A todo may have: description, due date, priority, one or more tags, and a project. |
| FR-CAP-3 | 🔴 | The user can edit, complete/uncomplete, and delete a todo. |
| FR-CAP-4 | 🔴 | A "quick capture" entry point lets the user add a todo in the fewest possible steps from every client. |
| FR-CAP-5 | 🟠 | A todo may have a deadline (immovable date) distinct from a (movable) due date. |
| FR-CAP-6 | 🟠 | The user can set reminders on a todo. |
| FR-CAP-7 | 🟢 | The user can attach files/links to a todo. |
| FR-CAP-8 | 🟢 | Recurring todos (repeat schedules) are supported. |

## Notes & Ideas — `FR-NOTE`

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-NOTE-1 | 🔴 | The user can create free-form notes using Markdown. |
| FR-NOTE-2 | 🔴 | The user can create lightweight "ideas" (short, unstructured captures) distinct from full notes. |
| FR-NOTE-3 | 🔴 | The user can edit, organize (tag/project), and delete notes and ideas. |
| FR-NOTE-4 | 🟠 | Notes support checkbox/task-list syntax (`- [ ]`). |
| FR-NOTE-5 | 🟠 | Notes are full-text searchable. |
| FR-NOTE-6 | 🟢 | Notes support links/backlinks between notes. |

## Auto-extraction of todos — `FR-EXTRACT`

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-EXTRACT-1 | 🔴 | DevTriage detects action items within notes/ideas and surfaces them as candidate todos. |
| FR-EXTRACT-2 | 🔴 | At minimum, Markdown checkbox lines (`- [ ]`) are recognized as action items. |
| FR-EXTRACT-3 | 🔴 | Each extracted todo links back to its source note and position. |
| FR-EXTRACT-4 | 🔴 | Extraction mode is configurable per user: **auto-create** vs **suggest-and-confirm**. |
| FR-EXTRACT-5 | 🟠 | Heuristic detection of imperative/action sentences beyond explicit checkboxes. |
| FR-EXTRACT-6 | 🟠 | Editing a source note keeps already-extracted todos in sync where possible (e.g. checking a box in the note completes the todo and vice-versa). |
| FR-EXTRACT-7 | 🟢 | Optional ML/LLM-assisted extraction, runnable locally / under user control (see open questions). |

## Integrations / assigned-item collection — `FR-INT`

> Detailed behavior is specified in [03 — Integration Requirements](03-integration-requirements.md).

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-INT-1 | 🔴 | The user can connect a GitHub account/credential. |
| FR-INT-2 | 🔴 | The user can connect a Jira account/credential. |
| FR-INT-3 | 🔴 | DevTriage collects GitHub **issues assigned to the user**. |
| FR-INT-4 | 🔴 | DevTriage collects GitHub **pull requests authored by or requesting review from the user**. |
| FR-INT-5 | 🔴 | DevTriage collects Jira **issues assigned to the user**. |
| FR-INT-6 | 🔴 | Collected external items appear in the unified inbox alongside personal todos. |
| FR-INT-7 | 🔴 | Items sync periodically and can be refreshed on demand. |
| FR-INT-8 | 🔴 | Each external item retains a deep link back to its source (GitHub/Jira URL). |
| FR-INT-9 | 🟠 | When an external item is closed/resolved at the source, its DevTriage representation reflects that on the next sync. |
| FR-INT-10 | 🟠 | The user can scope collection (e.g. specific repos/orgs, Jira projects/JQL). |
| FR-INT-11 | 🟢 | Limited write-back (e.g. mark Jira done / close GitHub issue) — pending decision. |
| FR-INT-12 | 🟢 | Pluggable connector model so further sources can be added later. |

## Unified triage — `FR-TRIAGE`

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-TRIAGE-1 | 🔴 | A single view combines personal todos, extracted todos, and external items. |
| FR-TRIAGE-2 | 🔴 | The user can filter by source, project, tag, priority, due date, and status. |
| FR-TRIAGE-3 | 🔴 | The user can sort and group items (e.g. by source, priority, due date). |
| FR-TRIAGE-4 | 🔴 | The user can search across all items. |
| FR-TRIAGE-5 | 🔴 | The user can complete/dismiss an item from the unified view. |
| FR-TRIAGE-6 | 🟠 | The user can snooze/defer an item to reappear later. |
| FR-TRIAGE-7 | 🟠 | The user can assign an external item to a local project and/or add local tags and notes to it without altering the source. |
| FR-TRIAGE-8 | 🟢 | Saved/custom views (e.g. "Today", "Needs review"). |

## Organization — `FR-ORG`

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-ORG-1 | 🔴 | Items can be organized into projects. |
| FR-ORG-2 | 🔴 | Items can be labeled with tags. |
| FR-ORG-3 | 🔴 | Todos have priority levels. |
| FR-ORG-4 | 🟠 | Projects can have sections/sub-groupings. |

## Sync & multi-device — `FR-SYNC`

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-SYNC-1 | 🔴 | Web, iOS, and Android clients operate on the same account data via a shared backend API. |
| FR-SYNC-2 | 🔴 | Changes on one client become visible on others after sync. |
| FR-SYNC-3 | 🟠 | Mobile clients support offline capture and reconcile when back online. |
| FR-SYNC-4 | 🟢 | Real-time/push updates (vs poll-based) to clients. |

## Settings, accounts & auth — `FR-SET`

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-SET-1 | 🔴 | The user can authenticate to DevTriage. |
| FR-SET-2 | 🔴 | The user can add, test, and revoke external integration credentials. |
| FR-SET-3 | 🔴 | Integration credentials are stored securely (see NFR-SEC). |
| FR-SET-4 | 🟠 | The user can configure sync frequency and extraction mode. |
| FR-SET-5 | 🟠 | The user can export their data. |

## Traceability

- Use cases → requirements mapping lives in [01 — Personas & Use Cases](01-personas-and-use-cases.md).
- Acceptance criteria for the highest-priority requirements are in [07 — User Stories](07-user-stories.md).
