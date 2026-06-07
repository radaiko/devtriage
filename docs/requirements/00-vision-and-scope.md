# 00 — Vision & Scope

## Problem

Developers and technical workers spread their "things to do" across many systems:

- Personal todos and reminders live in notes apps, sticky notes, or their head.
- Ideas and meeting notes contain action items that are never turned into tasks.
- Real work is assigned to them inside **GitHub** (issues, PRs, review requests) and
  **Jira** (tickets) — each with its own inbox and notification model.

The result is context-switching, missed work, and no single place that answers the
question: *"What do I actually need to do?"*

## Vision

**DevTriage is the single surface where a developer captures personal work and
triages everything assigned to them.** Manual todos, notes, and ideas live next to
issues, pull requests, and tickets automatically collected from external systems —
viewable and actionable on web, iOS, and Android.

## Goals

| # | Goal |
| --- | --- |
| G1 | Provide one fast place to capture todos, notes, and ideas. |
| G2 | Automatically surface action items hidden inside notes/ideas as todos. |
| G3 | Collect everything **assigned to me** (issues, PRs, tasks) from GitHub and Jira into one inbox. |
| G4 | Let the user triage all of it from a single, consistent view across devices. |
| G5 | Respect user data ownership and minimize security/supply-chain risk. |

## Success criteria

- A user can capture a thought in **under 3 seconds** on any client (NFR-PERF).
- After connecting GitHub and Jira, **100% of items assigned to the user** appear in
  the unified inbox within one sync cycle (FR-INT).
- Action items written in notes are detected and offered as todos without manual
  re-entry (FR-EXTRACT).
- The same data is consistently available on web, iOS, and Android.

## In scope (initial product)

- Manual todos, notes, and ideas (capture, edit, organize, complete).
- Auto-extraction of todos from notes/ideas.
- Read collection of assigned items from GitHub and Jira into a unified inbox.
- Triage actions over all items (prioritize, snooze, complete, group, search).
- Web client + native iOS (Swift) and Android (Kotlin) clients.
- Single-user / self-hostable deployment.

## Out of scope (initially)

- Team collaboration features (shared projects, assigning work to others).
- Writing back to GitHub/Jira beyond minimal status reflection (see open questions).
- Integrations beyond GitHub and Jira (GitLab, Linear, Asana, email, calendars) —
  the architecture should not preclude them, but they are not in the first release.
- Time tracking, sprint planning, analytics dashboards.
- Real-time multi-user editing.

## Guiding principles

1. **Capture must be frictionless.** If it's slower than a sticky note, it loses.
2. **External systems are the source of truth** for items they own; DevTriage is a
   triage layer over them, not a replacement.
3. **Minimal, auditable dependencies.** Security and supply-chain risk are first-class
   concerns (see [NFR-DEP](04-non-functional-requirements.md#dependency--supply-chain-policy-nfr-dep)).
4. **The user owns their data and credentials.**

## Related documents

- [01 — Personas & Use Cases](01-personas-and-use-cases.md)
- [02 — Functional Requirements](02-functional-requirements.md)
- [09 — Open Questions](09-open-questions.md)
