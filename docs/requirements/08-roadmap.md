# 08 — Roadmap (Phased)

A suggested sequencing of work **once requirements are signed off**. Phases are
outcome-oriented; dates are intentionally omitted until the stack is locked.

## Phase 0 — Requirements & decisions *(current)*

- ✅ Vision, scope, personas, functional & non-functional requirements documented.
- ⬜ Resolve [open questions](09-open-questions.md): backend language, web approach,
  auth mechanisms, storage, license.
- **Exit criteria:** requirements approved; firm tech decisions recorded in
  [06 — Architecture](06-architecture-and-tech-decisions.md).

## Phase 1 — Core capture (backend + one client)

- Backend skeleton: API, storage, auth (FR-SET-1).
- Todos CRUD (FR-CAP-1..4) and Notes/Ideas (FR-NOTE-1..3).
- First client (web) with quick capture and a basic list.
- **Exit criteria:** a user can capture and manage todos/notes on web, persisted via
  the backend.

## Phase 2 — Auto-extraction

- Checkbox extraction from notes (FR-EXTRACT-1..4) with suggest/auto modes.
- Source linkage and basic sync of completion state (FR-EXTRACT-3/6).
- **Exit criteria:** action items in notes reliably become linked todos.

## Phase 3 — Integrations (the differentiator)

- Connector framework (NFR-MNT-1, FR-INT-12).
- GitHub connector (INT-GH-1..3) and Jira connector (INT-JIRA-1).
- Unified inbox combining todos + external items (FR-TRIAGE-1).
- Credential storage & security (NFR-SEC-1..4).
- **Exit criteria:** assigned GitHub/Jira items appear and stay in sync in one inbox.

## Phase 4 — Triage power

- Filter/sort/group/search across sources (FR-TRIAGE-2..4).
- Snooze/defer and local overlays on external items (FR-TRIAGE-6/7).
- Status reflection from sources (FR-INT-9, INT-COM-8).
- **Exit criteria:** the inbox is genuinely usable as a daily triage surface.

## Phase 5 — Native mobile

- iOS (Swift/SwiftUI) and Android (Kotlin/Compose) clients on the shared API.
- Offline capture & sync (FR-SYNC-3, NFR-OFF-1); secure secret storage (NFR-SEC-3).
- **Exit criteria:** capture and triage work on both native platforms.

## Phase 6 — Hardening & polish

- Dependency vulnerability scanning in CI (NFR-DEP-5).
- Data export (FR-SET-5, NFR-PRIV-3), accessibility pass (NFR-A11Y), observability
  (NFR-OBS).
- Self-host packaging (single binary/container, NFR-PORT-1).

## Later / candidate (not committed)

- Real-time push updates (FR-SYNC-4).
- Limited write-back to sources (FR-INT-11).
- Additional connectors (GitLab, Linear, etc.) via the connector model.
- LLM-assisted extraction, user-controlled (FR-EXTRACT-7).
