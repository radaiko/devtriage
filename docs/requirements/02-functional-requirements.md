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
| FR-EXTRACT-4 | 🔴 | Extraction mode is configurable per user: **auto-create** vs **suggest-and-confirm**. **Default = confidence-based hybrid** (OQ-12): auto-create from explicit checkboxes, suggest-and-confirm for fuzzy heuristic detections. |
| FR-EXTRACT-5 | 🟠 | Heuristic detection of imperative/action sentences beyond explicit checkboxes. |
| FR-EXTRACT-6 | 🟠 | Editing a source note keeps already-extracted todos in sync where possible (e.g. checking a box in the note completes the todo and vice-versa). |
| FR-EXTRACT-7 | 🟢 | Optional ML/LLM-assisted extraction, runnable locally / under user control (see open questions). |

## Integrations / assigned-item collection — `FR-INT`

> Detailed behavior is specified in [03 — Integration Requirements](03-integration-requirements.md).
> Collection is **client-side**: each app polls GitHub/Jira directly with tokens kept
> on-device; the server never sees tokens or fetched items.

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-INT-0 | 🔴 | Integration polling happens **client-side**; tokens are stored on-device (and optionally in encrypted BYO storage — [OQ-25](09-open-questions.md)), never on the server. |
| FR-INT-0b | 🔴 | On **web**, providers browsers can't call directly (Jira, WebDAV) are reached via a **local companion app** on the user's machine that holds the token locally; provider tokens never transit the DevTriage server ([OQ-8a](09-open-questions.md)). |
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

> Architecture: clients are **local-first** and sync through the user's **own storage
> (BYO)**; the hosted server stores no data. See
> [06 — Architecture](06-architecture-and-tech-decisions.md).

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-SYNC-1 | 🔴 | Web, iOS, and Android clients share the user's data by syncing through the user's connected **bring-your-own storage** — **not** a server-side database. |
| FR-SYNC-2 | 🔴 | Changes on one client become visible on others after a sync cycle through BYO storage. |
| FR-SYNC-3 | 🔴 | All clients are local-first: they work offline against a local store and reconcile when connectivity returns. |
| FR-SYNC-4 | 🔴 | Concurrent edits from multiple devices are merged deterministically with no silent data loss (strategy is [OQ-23](09-open-questions.md)). |
| FR-SYNC-5 | 🔴 | Data written to BYO storage is **encrypted client-side** so the storage provider cannot read it ([NFR-SEC](04-non-functional-requirements.md), [OQ-24](09-open-questions.md)). |
| FR-SYNC-6 | 🟠 | A thin **server-side E2EE sync-coordination** layer may provide change notifications, version coordination, and device key-exchange to speed up sync — storing **no readable content** ([06](06-architecture-and-tech-decisions.md), [OQ-26](09-open-questions.md)). |
| FR-SYNC-7 | 🟢 | Full real-time/push updates building on FR-SYNC-6. |

## Settings, accounts & auth — `FR-SET`

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-SET-0 | 🔴 | The user can sign in **passwordlessly** via **passkey**, **Sign in with Google**, or **Sign in with GitHub**. There is **no password-based account creation**; the account is provisioned automatically on first sign-in. It holds only minimal identity (public key / provider subject id + timestamps) and never user content (see [NFR-PRIV-1](04-non-functional-requirements.md)). |
| FR-SET-0a | 🔴 | The account/login flow **clearly explains why an account is required** — (1) to coordinate end-to-end-encrypted cross-device sync, (2) to protect the sync-coordination service from abuse, and (3) for user-count metrics — and **reassures that no notes/todos/documents, tokens, or fetched items are stored on the server**. |
| FR-SET-1 | 🔴 | The user can connect a **bring-your-own storage** backend to enable cross-device sync, and test/disconnect it. |
| FR-SET-2 | 🔴 | The user can add, test, and revoke external integration credentials. |
| FR-SET-3 | 🔴 | Integration credentials and BYO-storage credentials are stored securely **on-device** (platform secure store) and never sent to the DevTriage server (see NFR-SEC). |
| FR-SET-4 | 🟠 | The user can configure sync frequency and extraction mode. |
| FR-SET-5 | 🟠 | The user can export their data (data already lives in their own storage, but a portable export is provided). |
| FR-SET-6 | 🔴 | The DevTriage login must not require the server to store any user content — only minimal account/identity data (see [NFR-PRIV-1](04-non-functional-requirements.md)). |

## Traceability

- Use cases → requirements mapping lives in [01 — Personas & Use Cases](01-personas-and-use-cases.md).
- Acceptance criteria for the highest-priority requirements are in [07 — User Stories](07-user-stories.md).
