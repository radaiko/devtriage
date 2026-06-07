# CLAUDE.md

Guidance for AI assistants (and humans) working in this repository.

## What this repo is

**DevTriage** — a universal developer tool. The core product is a **unified todo +
notes tool** that also **collects work assigned to the user from GitHub and Jira**
(issues, pull requests, tasks) into a single triage surface, available on **web,
iOS, and Android**.

## Current phase: REQUIREMENTS FIRST (code comes later)

This is **deliberate sequencing, not a permanent ban** — implementation *will* happen
in later sessions. The current goal is to produce and refine clear, agreed
requirements before any implementation starts, so the repository currently contains
documentation only.

🚫 **Do not write application code until the user says the requirements phase is
done.** When asked to make changes in this phase, default to editing the documents
under [`docs/requirements/`](docs/requirements/). Once the user signals that
requirements are complete (and the open architecture decisions are resolved),
implementation work is expected and welcome.

## Hard constraints (do not violate without explicit sign-off)

- **Minimal dependency surface.** Avoid pulling in large third-party dependency
  trees, especially from the npm ecosystem — the user has explicitly flagged
  supply-chain attack risk. Prefer standard libraries and a small set of audited
  dependencies. Captured as `NFR-DEP`.
- **Native mobile only.** iOS = Swift/SwiftUI, Android = Kotlin/Jetpack Compose. Do
  not propose React Native / Flutter / other cross-platform JS runtimes for mobile.
- **The server stores no customer content.** DevTriage is a **hosted multi-tenant
  service** run by the project owner on a **Hetzner VM**, but the server persists **no
  documents/notes/todos, no integration tokens, and no fetched items**. It **may** hold
  only **E2EE / opaque sync-coordination metadata** (version pointers, change
  notifications, encrypted key-exchange) that it cannot read. Clients are
  **local-first**; content syncs through the user's **own storage (BYO)** with
  client-side encryption; GitHub/Jira are **polled client-side** (tokens stay
  on-device). Do not introduce a server-side store of readable user content. Captured
  as `NFR-PRIV`.

## How requirements are organized

- Requirements live in `docs/requirements/`, numbered `00`–`09`.
- Functional requirements use IDs like `FR-CAP-1`; non-functional use `NFR-SEC-1`.
- When you add or change a requirement, keep IDs stable; append new ones rather than
  renumbering, and update cross-references.
- Unresolved decisions go in `docs/requirements/09-open-questions.md`. Don't silently
  pick a side on an open question inside the other docs — note it as proposed and
  link to the open question.

## Working agreements

- Keep documents concise and skimmable; prefer tables and numbered lists.
- Cross-link requirement IDs so traceability is easy.
- The architecture/tech-stack doc records **proposals with rationale**, not locked
  decisions — flag anything still open.

## Git

- Develop on the branch assigned for the task; commit with clear messages; push when
  changes are complete.
- Do not open a pull request unless explicitly asked.
