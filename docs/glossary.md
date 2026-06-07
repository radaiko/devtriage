# Glossary

Shared vocabulary for DevTriage. Keep terms here consistent across all documents.

| Term | Meaning |
| --- | --- |
| **DevTriage** | This product: a universal developer tool unifying personal todos/notes with assigned work collected from GitHub and Jira. |
| **Todo** | A user-actionable task. May be created manually or extracted from a note/idea. |
| **Note** | Free-form Markdown content (e.g. meeting/design notes). May contain action items. |
| **Idea** | A lightweight, short, unstructured capture — smaller than a Note. |
| **Action item** | A task-like line inside a note/idea (e.g. a `- [ ]` checkbox) that can become a Todo. |
| **Auto-extraction / Auto-collection** | Detecting action items inside notes/ideas and surfacing them as Todos. |
| **External item** | A GitHub or Jira item assigned to the user, collected by a connector and shown in the inbox. A read model — the external system stays the source of truth. |
| **Connector** | A server-side module that authenticates to an external system and syncs external items (e.g. the GitHub connector, the Jira connector). |
| **Unified inbox** | The single view combining personal Todos and External items for triage. A view, not a stored entity. |
| **Triage** | The act of prioritizing, snoozing, grouping, completing, or dismissing items in the inbox. |
| **Local overlay** | User-applied metadata (tags, priority, snooze, notes) on an External item that does not modify the source. |
| **Sync** | Pulling the latest external items from a source (scheduled or on-demand). |
| **Write-back** | Sending changes from DevTriage back to a source (e.g. closing a GitHub issue). Out of initial scope. |
| **Local-first** | Architecture where each client holds its own data and is the working source of truth, functioning offline and syncing later. |
| **BYO storage (bring-your-own)** | The user's own storage backend (e.g. WebDAV, S3, a private Git repo, a cloud drive) that DevTriage clients sync content through. Content is encrypted client-side. |
| **Storage adapter** | The client-side abstraction over a specific BYO storage backend. |
| **Sync coordination (E2EE)** | Optional thin server-side layer holding only opaque/encrypted metadata (version pointers, change notifications, key-exchange) to speed sync — never readable content. |
| **CORS proxy** | The stateless server component that forwards web-client requests to provider APIs that disallow browser-origin calls; stores nothing. |
| **Hosted service** | DevTriage is operated by the project owner on a Hetzner VM (not self-hosted by end users), yet the server stores no customer content. |
| **Account** | A required, minimal identity record on the server (id + auth identity + timestamps) used for login and user-count metrics. Holds no user content, tokens, or fetched items. |
| **FR-* / NFR-*** | Functional / Non-functional requirement IDs used for traceability. |
| **OQ-*** | Open question IDs in [09 — Open Questions](requirements/09-open-questions.md). |
