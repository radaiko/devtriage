# 03 — Integration Requirements

This document details how DevTriage collects items **assigned to the user** from
external systems. It expands on [`FR-INT`](02-functional-requirements.md#integrations--assigned-item-collection-fr-int).

## General model

- Each integration is a **connector** that runs **client-side** (inside the web, iOS,
  and Android apps), authenticates as the user, and pulls items on a schedule (and on
  demand). **The hosted server never holds tokens or fetched items.**
- **Mobile** apps call GitHub/Jira **directly**. The **web** app routes through the
  stateless Hetzner **CORS proxy** where a provider disallows browser-origin calls; the
  proxy forwards but does not store (see [OQ-8a](09-open-questions.md)).
- Collected items are stored as **external items** (a read model) in the client's local
  store and synced via the user's BYO storage. DevTriage is a triage layer; the external
  system remains the source of truth (see [05 — Data Model](05-data-model.md)).
- Connectors are **read-only** by default. Any write-back is a separate, explicitly
  opt-in capability (see open questions).
- Sync is **incremental** where the source API allows, to respect rate limits.
- **Background collection while the app is closed is limited** (especially on web),
  since polling is client-side — an accepted trade-off of the "server stores nothing"
  decision.

## Common requirements (all connectors)

| ID | Priority | Requirement |
| --- | --- | --- |
| INT-COM-1 | 🔴 | Authenticate using a user-supplied credential (token) per source. |
| INT-COM-2 | 🔴 | Provide a "test connection" action that verifies the credential and identity. |
| INT-COM-3 | 🔴 | Resolve and cache the user's identity on the source (to determine "assigned to me"). |
| INT-COM-4 | 🔴 | Run a periodic sync and an on-demand refresh. |
| INT-COM-5 | 🔴 | De-duplicate items so re-syncing does not create duplicates (stable external IDs). |
| INT-COM-6 | 🔴 | Surface sync errors (auth failure, rate limit) to the user without losing existing data. |
| INT-COM-7 | 🟠 | Respect and back off on API rate limits. |
| INT-COM-8 | 🟠 | Reflect source status changes (open → closed/resolved) on next sync. |
| INT-COM-9 | 🟠 | Allow the user to scope what is collected. |

## GitHub connector — `INT-GH`

**Authentication (decided — OQ-6): Personal Access Token.** Prefer a **fine-grained
PAT** with least-privilege read scopes (issues, pull requests, metadata) for the repos/
orgs the user selects; a **classic PAT** is allowed when the user wants broad
"everything assigned to me across all repos" reach. The token is entered by the user and
stored on-device (NFR-SEC-1/3) — **no OAuth flow / server callback needed**. Web calls
GitHub directly where CORS allows (GitHub's REST API generally supports browser CORS);
otherwise via the proxy. Token expiry/revocation is surfaced per INT-COM-6. Supports
github.com; GitHub Enterprise is a future consideration.

**What "assigned to me" means — collect:**

| ID | Priority | Item set |
| --- | --- | --- |
| INT-GH-1 | 🔴 | Issues **assigned to** the authenticated user. |
| INT-GH-2 | 🔴 | Pull requests **requesting review** from the user. |
| INT-GH-3 | 🔴 | Pull requests **authored by** the user that are open. |
| INT-GH-4 | 🟠 | Issues/PRs where the user is **@-mentioned**. |
| INT-GH-5 | 🟠 | Items **assigned to** the user across all accessible repos/orgs, with optional scoping to selected repos/orgs. |

**Fields to capture per item:** source type (issue/PR), title, number, repository,
state (open/closed/merged/draft), URL, labels, assignees, review state (for PRs),
created/updated timestamps.

**Notes:**

- Prefer GitHub's search/issues APIs that already filter by assignee/review-requested
  to minimize calls.
- Distinguish PR sub-states relevant to triage (e.g. *review requested* vs *changes
  requested* vs *approved*) where feasible.

## Jira connector — `INT-JIRA`

**Authentication (decided — OQ-7): Atlassian API token + account email (Basic auth).**
The user enters their email + API token, stored on-device (NFR-SEC-1/3) — **no OAuth
flow / server callback needed**. Jira Cloud's REST API generally does **not** allow
browser CORS, so the **web** client calls via the Hetzner proxy ([OQ-28](09-open-questions.md));
mobile calls directly. Jira Cloud is the initial target; Jira Server/Data Center (which
also supports PATs) is a future consideration.

| ID | Priority | Item set |
| --- | --- | --- |
| INT-JIRA-1 | 🔴 | Issues where the current user is the **assignee**, in non-done statuses. |
| INT-JIRA-2 | 🟠 | Issues the user **reports/watches** (optional, user-configurable). |
| INT-JIRA-3 | 🟠 | User-supplied **JQL** to define exactly what is collected. |
| INT-JIRA-4 | 🟠 | Scope collection to selected projects. |

**Fields to capture per item:** issue key, summary, project, issue type, status (and
status category: to-do/in-progress/done), priority, URL, assignee, due date,
created/updated timestamps.

**Notes:**

- Default collection query should resemble `assignee = currentUser() AND statusCategory
  != Done`, overridable by user JQL (INT-JIRA-3).
- Map Jira status categories to DevTriage's notion of open/closed for INT-COM-8.

## Mapping external items into DevTriage

| External concept | DevTriage representation |
| --- | --- |
| GitHub issue / PR, Jira issue | **External item** (read model) shown in the unified inbox |
| Source state (open/closed/merged/resolved) | Normalized open/closed status |
| Title / summary | Item title |
| URL | Deep link (FR-INT-8) |
| Labels / components | Optional mapping to local tags (user-controlled) |

The user **may** layer local metadata on an external item — local tags, project,
priority, snooze, notes — without modifying the source (`FR-TRIAGE-7`).

## Out of scope for first release

- Bi-directional sync / write-back beyond minimal status reflection.
- Sources other than GitHub and Jira (the connector model keeps the door open —
  `FR-INT-12`).
- GitHub Enterprise Server and Jira Server/Data Center (future).

## Open integration questions

See [09 — Open Questions](09-open-questions.md): auth mechanism per source, whether
to support write-back, and the web CORS / transient token pass-through question
([OQ-8a](09-open-questions.md)).
