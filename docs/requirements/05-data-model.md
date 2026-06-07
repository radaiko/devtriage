# 05 — Data Model (Conceptual)

This is a **conceptual** model to anchor the requirements — not a database schema.
Concrete schema/storage decisions are deferred (see
[06 — Architecture](06-architecture-and-tech-decisions.md) and
[09 — Open Questions](09-open-questions.md)).

## Where data lives (important)

There is **no server-side database**. The entities below live:

- in each client's **local store** (the working source of truth — local-first), and
- synced through the user's **bring-your-own storage**, **encrypted client-side**.

The hosted Hetzner server stores none of it. See
[06 — Architecture](06-architecture-and-tech-decisions.md).

## Core entities

### User
The account that owns everything. Single-user focus initially.
- `id`, `display name`, auth credentials (see NFR-SEC).

### Todo
A user-actionable task.
- `id`, `title`, `description`, `status` (open/done), `priority`, `due date`,
  `deadline` (optional), `created/updated`, `completed at`.
- Optional links: `project`, `tags[]`.
- **Origin**: `manual` | `extracted` | (potentially) `promoted-from-external`.
- If extracted: reference to the **source note** and position (FR-EXTRACT-3).

### Note / Idea (one entity — OQ-14)
Free-form text is **a single entity with a `kind` flag** (`note | idea`), not two
separate entities. They share storage, sync, encryption, search, and action-item
extraction; the difference is UX/intent (a *note* is a fuller Markdown document; an
*idea* is a quick, short, usually title-less capture).
- `id`, `kind` (`note | idea`), `title` (optional), `body` (Markdown), `created/updated`.
- Optional links: `project`, `tags[]`.
- May contain action items that yield extracted Todos (FR-EXTRACT).
- **Promotion** is just changing state: idea → note flips `kind`; idea/note → Todo uses
  the existing promotion path.

### ExternalItem (read model)
A GitHub/Jira item assigned to the user, collected by a connector.
- `id` (DevTriage-local), `source` (github | jira), `external id` (stable),
  `type` (issue | pull_request | jira_issue), `title`, `url`, `status`
  (normalized open/closed), `source status` (raw), `priority`, `timestamps`.
- Source-specific fields: repo, number, review state (GitHub); key, project, issue
  type, status category (Jira).
- **Local overlay** (user-applied, does not change the source — FR-TRIAGE-7):
  `local tags[]`, `local project`, `local priority`, `snooze until`, `local note`,
  `dismissed`.

### Project
A grouping for todos, notes, and (via overlay) external items.
- `id`, `name`, optional `sections[]`.

### Tag
A label applied across todos, notes, ideas, and external-item overlays.
- `id`, `name`.

### IntegrationConnection
A configured link to an external system. Lives **on-device** (token in the platform
secure store), optionally mirrored as client-encrypted data in BYO storage ([OQ-25](09-open-questions.md)).
- `id`, `source` (github | jira), `credential` (on-device / encrypted — NFR-SEC-1),
  `scope` config (repos/orgs or projects/JQL), `last synced at`, `status`/last error.

### SyncState (E2EE coordination metadata — server-side)
Per-user sync-coordination metadata the hosted server stores to improve sync — strictly
**opaque, never document content** (schema fixed in [06 — Architecture](06-architecture-and-tech-decisions.md), OQ-26):
- **Sync generation counter** + opaque token (cheap "is there anything new"; per-item
  version vectors live in the encrypted BYO payload, not here).
- **Wake/notify channel** — contentless "gen bumped" signal.
- **Device registry** — opaque device id + **public key** + timestamps; human labels
  client-encrypted.
- **Key-exchange envelope mailbox** — ephemeral ciphertext (wrapped DEK), deleted after
  pickup.
- The server reads **none** of the underlying content; it sees ciphertext, public keys,
  and opaque counters only. No item-level metadata; no change history.

## Relationships (text view)

```
User 1──* Todo
User 1──* Note/Idea 1──* Todo   (one entity w/ kind flag; extracted todos link to source)
User 1──* ExternalItem          (collected from connectors)
User 1──* IntegrationConnection 1──* ExternalItem
Project 1──* Todo / Note/Idea   (and external items via overlay)
Tag *──* Todo / Note-Idea / ExternalItem
```

## The "Unified Inbox"

The unified inbox (FR-TRIAGE-1) is a **view**, not a stored entity: it is the union
of open Todos (manual + extracted) and non-dismissed ExternalItems, with shared
filtering/sorting/grouping applied across both.

## Key invariants

- An extracted Todo always references a valid source Note (or records that the source
  was deleted) — FR-EXTRACT-3.
- ExternalItems are uniquely keyed by `(source, external id)` to prevent duplicates on
  re-sync — INT-COM-5.
- Deleting/Failing a sync must not delete previously collected ExternalItems —
  NFR-REL-2.
- Local overlay data on an ExternalItem is never pushed to the source unless explicit
  write-back is enabled (out of scope initially).
