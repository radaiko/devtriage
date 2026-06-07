# DevTriage

> A universal developer tool — one place to capture what's on your mind and triage
> everything that's been assigned to you.

DevTriage unifies your **personal todos and notes** with the **work items already
assigned to you across the tools you use** (GitHub, Jira). Instead of checking five
inboxes, you get a single, fast triage surface on **web, iOS, and Android**.

## What it does

- **Unified todo + notes.** Capture todos, free-form notes, and ideas in one place.
- **Auto-collected todos.** Action items written inside notes and ideas are detected
  and surfaced as todos, linked back to their source.
- **One inbox for assigned work.** Issues, pull requests, and tasks assigned to you
  (or requesting your review) are pulled in from **GitHub** and **Jira** and shown
  alongside your personal todos.
- **Triage, don't juggle.** Prioritize, snooze, group, and complete items from a
  single view — regardless of which system they came from.
- **Everywhere you work.** Web app plus **native** mobile clients (Swift / Kotlin).

## Project status

🟡 **Requirements phase — no code yet.** (Implementation comes next, once requirements
are signed off.)

This repository currently contains **requirements documentation only**. The goal of
this phase is to agree on *what* DevTriage should do and the constraints it must
respect before any implementation begins — it's deliberate sequencing, not a permanent
documentation-only project.

Start here:

- [`docs/requirements/00-vision-and-scope.md`](docs/requirements/00-vision-and-scope.md) — why this exists, what's in/out of scope
- [`docs/requirements/02-functional-requirements.md`](docs/requirements/02-functional-requirements.md) — the numbered feature requirements
- [`docs/requirements/09-open-questions.md`](docs/requirements/09-open-questions.md) — decisions still to be made

## Documentation map

| Document | Contents |
| --- | --- |
| [00 — Vision & Scope](docs/requirements/00-vision-and-scope.md) | Problem, goals, non-goals, scope boundaries |
| [01 — Personas & Use Cases](docs/requirements/01-personas-and-use-cases.md) | Who uses it and the core flows |
| [02 — Functional Requirements](docs/requirements/02-functional-requirements.md) | Numbered functional requirements (FR-*) |
| [03 — Integration Requirements](docs/requirements/03-integration-requirements.md) | GitHub & Jira collection in detail |
| [04 — Non-Functional Requirements](docs/requirements/04-non-functional-requirements.md) | Security, privacy, performance, dependency policy (NFR-*) |
| [05 — Data Model](docs/requirements/05-data-model.md) | Core entities and relationships |
| [06 — Architecture & Tech Decisions](docs/requirements/06-architecture-and-tech-decisions.md) | Proposed stack and the reasoning/constraints |
| [07 — User Stories](docs/requirements/07-user-stories.md) | Stories with acceptance criteria |
| [08 — Roadmap](docs/requirements/08-roadmap.md) | Phased milestones |
| [09 — Open Questions](docs/requirements/09-open-questions.md) | Unresolved decisions |
| [Glossary](docs/glossary.md) | Shared vocabulary |

## Key constraints (agreed)

1. **Minimal dependency surface.** Given recent supply-chain attacks, DevTriage
   avoids large third-party dependency trees (especially the npm ecosystem). Prefer
   standard libraries, few and well-audited dependencies. See
   [NFR-DEP](docs/requirements/04-non-functional-requirements.md#dependency--supply-chain-policy-nfr-dep).
2. **Native mobile.** Mobile clients are native: **Swift/SwiftUI** (iOS) and
   **Kotlin/Jetpack Compose** (Android) — not a cross-platform JS framework.
3. **The server stores no customer content.** DevTriage is a hosted service (run on a
   Hetzner VM), but the server holds **no documents/notes/todos, tokens, or fetched
   items** — only **minimal account records** (for login + user counts), aggregate
   metrics, and at most **E2EE/opaque sync-coordination metadata** it cannot read.
   Clients are **local-first**, content syncs through the user's **own storage (BYO)**
   with client-side encryption, and GitHub/Jira are **polled client-side**. See
   [NFR-PRIV](docs/requirements/04-non-functional-requirements.md#privacy--data-ownership-nfr-priv)
   and [06 — Architecture](docs/requirements/06-architecture-and-tech-decisions.md).

## License

**Source-available under the Business Source License (BSL 1.1)** (OQ-19) — the source is
readable and self-hostable, but offering DevTriage as a competing hosted service is not
permitted. The BSL parameters (Change Date, Change License, Additional Use Grant) are
being finalized before first publish ([OQ-30](docs/requirements/09-open-questions.md));
the `LICENSE` file will be added then.
