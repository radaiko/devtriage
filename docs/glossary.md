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
| **Self-hosting** | Running the DevTriage backend on infrastructure the user controls. |
| **FR-* / NFR-*** | Functional / Non-functional requirement IDs used for traceability. |
| **OQ-*** | Open question IDs in [09 — Open Questions](requirements/09-open-questions.md). |
