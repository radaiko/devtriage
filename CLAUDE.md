# CLAUDE.md

Guidance for AI assistants (and humans) working in this repository.

## What this repo is

**DevTriage** — a universal developer tool. The core product is a **unified todo +
notes tool** that also **collects work assigned to the user from GitHub and Jira**
(issues, pull requests, tasks) into a single triage surface, available on **web,
iOS, and Android**.

## Current phase: IMPLEMENTATION (requirements signed off 2026-06-07)

✅ **The requirements phase is complete** — the owner signed off on 2026-06-07. All
gating decisions are made; the remaining [open questions](docs/requirements/09-open-questions.md)
(OQ-15 LLM features; OQ-25/26a/27/29 implementation details) are deferred to the phase
they belong to and do **not** block coding.

**Implementation is now expected and welcome.** Build per the agreed plan:
- Sequencing is in [08 — Roadmap](docs/requirements/08-roadmap.md) (start at Phase 1).
- Decided stack/architecture is in [06 — Architecture](docs/requirements/06-architecture-and-tech-decisions.md#decided).
- Keep requirements docs in sync as implementation reveals new detail; record new
  decisions back into [06] and resolve deferred OQs in [09] when their phase arrives.
- **Repo layout: monorepo** (OQ-20) — backend + companion (both Go, shared connector
  code) + web (TypeScript) + iOS (Swift) + Android (Kotlin) + docs.

The hard constraints below still apply at all times.

## Hard constraints (do not violate without explicit sign-off)

- **Minimal dependency surface.** Avoid pulling in large third-party dependency
  trees, especially from the npm ecosystem — the user has explicitly flagged
  supply-chain attack risk. Captured as `NFR-DEP`. In practice:
  - **Build small/utility things in-house** on native/standard APIs (e.g. native
    `fetch`, **not** axios; hand-rolled reactivity, routing, small parsing).
  - **Reserve external packages for genuinely large features** that are too much effort
    to reimplement (e.g. a rich text / Markdown editor).
  - **Version-aging rule:** when adding a dependency, pick the latest version that has
    **no known vulnerabilities** and was **released ≥1 month ago**.
  - **Pin exactly & update manually.** Pin to an exact version (no `^`/`~`/ranges),
    vendor it (commit + checksum). **No automated updates / update bots** — every bump
    is human-initiated, reviewed, and re-pinned.
- **Native mobile only.** iOS = Swift/SwiftUI, Android = Kotlin/Jetpack Compose. Do
  not propose React Native / Flutter / other cross-platform JS runtimes for mobile.
- **The server stores no customer content.** DevTriage is a **hosted multi-tenant
  service** run by the project owner on a **Hetzner VM**, but the server persists **no
  documents/notes/todos, no integration tokens, and no fetched items**. It stores only
  **minimal account/identity records** (id + auth identity + timestamps) for login and
  user-count metrics, **aggregate** usage metrics, and optionally **E2EE / opaque
  sync-coordination metadata** it cannot read. Clients are **local-first**; content
  syncs through the user's **own storage (BYO)** with client-side encryption;
  GitHub/Jira are **polled client-side** (tokens stay on-device). Do not introduce a
  server-side store of readable user content. Captured as `NFR-PRIV`.

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
