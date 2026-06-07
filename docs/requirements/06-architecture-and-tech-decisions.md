# 06 — Architecture & Tech Decisions

> **Status: PROPOSALS, not locked.** This document records candidate decisions with
> rationale so we can debate them. Anything still undecided is tracked in
> [09 — Open Questions](09-open-questions.md). Two constraints below are **firm**
> because the project owner stated them explicitly.

## Firm constraints

1. **Native mobile clients** — iOS in **Swift/SwiftUI**, Android in
   **Kotlin/Jetpack Compose**. No cross-platform JS runtimes (React Native, Flutter).
2. **Minimal dependency surface** — avoid large third-party dependency trees,
   **especially npm**. See [NFR-DEP](04-non-functional-requirements.md#dependency--supply-chain-policy-nfr-dep).

## Decided

| Decision | Choice | Date | Rationale (summary) |
| --- | --- | --- | --- |
| **Backend language** (OQ-1) | **Go** | 2026-06-07 | Std library covers HTTP server+client, JSON, crypto, `database/sql` → minimal deps (NFR-DEP); single static binary (NFR-PORT); goroutines fit I/O-bound integration polling. Rust would pull a larger async HTTP dep tree for an I/O-bound app; Kotlin would share a language with Android but lose the single-binary benefit. See details below. |

## High-level shape

```
            ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
            │   Web client │   │  iOS (Swift) │   │ Android (Kt) │
            └──────┬───────┘   └──────┬───────┘   └──────┬───────┘
                   │                  │                  │
                   └──────────  HTTPS / JSON API  ───────┘
                                      │
                              ┌───────▼────────┐
                              │   Backend API  │
                              │  (todos, notes,│
                              │   extraction,  │
                              │   triage)      │
                              └───┬───────┬────┘
                          sync    │       │   store
                       ┌──────────▼─┐  ┌──▼───────────┐
                       │ Connectors │  │   Database   │
                       │ GitHub/Jira│  │  (embedded)  │
                       └─────┬──────┘  └──────────────┘
                             │
                   external APIs (GitHub, Jira)
```

A **shared backend API** is the single source of truth for the three clients
(FR-SYNC-1). Connectors run server-side on a schedule and on demand.

## Backend — **decided: Go** (OQ-1)

**Why Go:**

- Compiles to a **single static binary** → trivial self-hosting (NFR-PORT-1).
- **Strong standard library** (HTTP server, JSON, crypto) → very few third-party
  deps, directly serving NFR-DEP. **No npm involved.**
- Excellent **concurrency** for polling multiple integrations and respecting rate
  limits (NFR-PERF-3, INT-COM-7).
- Mature, well-audited GitHub/Jira client options (or thin hand-rolled HTTP clients
  to keep deps minimal).

**Alternatives considered:** Rust (great safety/perf, but no stdlib HTTP → pulls an
async dep tree, and steeper velocity for an I/O-bound app), Kotlin/JVM (would share a
language with Android, but loses the single-binary benefit and needs a JVM/GraalVM),
Python/FastAPI (more and heavier deps, not a single binary), Node/TS (rejected —
conflicts with the npm dependency-avoidance constraint).

**Known caveat:** Go's `database/sql` needs a SQLite driver — either `mattn/go-sqlite3`
(mature, requires cgo / a C toolchain) or `modernc.org/sqlite` (pure-Go, larger
transpiled dependency). This is the one deliberate dependency the storage choice
introduces; still well within [NFR-DEP](04-non-functional-requirements.md#dependency--supply-chain-policy-nfr-dep).
The cgo-vs-pure-Go choice is deferred until implementation.

## Storage — proposed: **SQLite (embedded)**

- Single-file, zero-ops, ideal for single-user self-hosting (NFR-PORT-2).
- Full-text search available for notes (FR-NOTE-5) without extra services.
- Postgres remains an option for multi-user/scaled deployments later.

## API — proposed: **REST/JSON over HTTPS**, versioned

- Simple, language-agnostic — friendly to Swift/Kotlin/web clients.
- Documented contract, versioned (NFR-MNT-2).
- Real-time push (FR-SYNC-4) is a later enhancement; poll/refresh first.

## Web client — **decision needed (constraint-sensitive)**

The web client is where the npm-avoidance constraint bites hardest. Options:

| Option | Dependency posture | Trade-off |
| --- | --- | --- |
| **Server-rendered (Go templates) + light HTML/CSS, optional htmx** | Lowest — little/no JS build chain or npm | Less app-like interactivity |
| **Vanilla TypeScript, no framework, minimal/zero deps** | Low | More hand-written UI code |
| **Small framework (e.g. Svelte/Vue) with a tightly-controlled, audited dep set** | Higher (build tooling pulls npm) | Better DX, conflicts with NFR-DEP-2 |

**Proposed default:** start server-rendered + progressive enhancement to honor
NFR-DEP; revisit if interactivity needs grow. → see open questions.

## Mobile clients — firm

- **iOS:** Swift + SwiftUI; secrets in Keychain (NFR-SEC-3); offline capture
  (NFR-OFF-1) via local store synced to the API.
- **Android:** Kotlin + Jetpack Compose; secrets in Android Keystore; same offline
  model.
- Both consume the shared REST API; no shared business logic across native apps
  beyond the API contract (keeps each native and idiomatic).

## Connector design

- Each source (GitHub, Jira) is an isolated module implementing a common interface:
  `authenticate`, `testConnection`, `resolveIdentity`, `sync(incremental)` →
  normalized ExternalItems (NFR-MNT-1, FR-INT-12).
- Prefer thin HTTP clients over heavy SDKs to keep dependencies minimal (NFR-DEP).
- Scheduler triggers periodic sync; manual refresh hits the same path (INT-COM-4).

## Security posture (summary)

- Encrypt integration credentials at rest; never log them (NFR-SEC-1, NFR-SEC-4).
- TLS everywhere (NFR-SEC-2); platform secure stores on mobile (NFR-SEC-3).
- Pin and integrity-check every dependency (NFR-DEP-3); CI dependency scanning once
  code begins (NFR-DEP-5).

## What this buys us against the constraints

- **npm avoided** in the backend entirely; minimized/optional on web.
- **Native mobile** as required.
- **Self-hostable single binary + single-file DB** for data ownership (NFR-PRIV-1).

> Open architectural decisions are consolidated in
> [09 — Open Questions](09-open-questions.md).
