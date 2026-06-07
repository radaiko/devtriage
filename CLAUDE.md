# CLAUDE.md

Guidance for AI assistants (and humans) working in this repository.

## What this repo is

**DevTriage** — a universal developer tool. The core product is a **unified todo +
notes tool** that also **collects work assigned to the user from GitHub and Jira**
(issues, pull requests, tasks) into a single triage surface, available on **web,
iOS, and Android**.

## Current phase: REQUIREMENTS ONLY

🚫 **Do not write application code in this phase.** The repository deliberately
contains only documentation. The current goal is to produce and refine clear,
agreed requirements before any implementation starts.

When asked to make changes, default to editing the documents under
[`docs/requirements/`](docs/requirements/). Only start implementation work after the
user explicitly says the requirements phase is complete.

## Hard constraints (do not violate without explicit sign-off)

- **Minimal dependency surface.** Avoid pulling in large third-party dependency
  trees, especially from the npm ecosystem — the user has explicitly flagged
  supply-chain attack risk. Prefer standard libraries and a small set of audited
  dependencies. Captured as `NFR-DEP`.
- **Native mobile only.** iOS = Swift/SwiftUI, Android = Kotlin/Jetpack Compose. Do
  not propose React Native / Flutter / other cross-platform JS runtimes for mobile.
- **Self-hostable / user owns data.** Treat credentials and synced data as sensitive
  and user-controlled.

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
