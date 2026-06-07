# 09 — Open Questions

Decisions still to be made. Each should be resolved before (or early in) the relevant
phase. Owner = project owner unless stated.

## Architecture & stack

| # | Question | Notes / leaning |
| --- | --- | --- |
| OQ-1 | ✅ **RESOLVED — Backend language = Go** (2026-06-07). | Chosen for minimal deps (std-lib HTTP/JSON/crypto/sql), single static binary, and goroutine concurrency for integration polling. Recorded in [06 — Decided](06-architecture-and-tech-decisions.md#decided). One known caveat: SQLite driver (cgo vs pure-Go) deferred to implementation. |
| OQ-2 | ✅ **RESOLVED — Platform-first vanilla TypeScript web client, built with esbuild, plus at most a couple of tiny vendored zero-transitive-dependency libs** (2026-06-07). | Lean on native browser APIs (Web Components, IndexedDB, Web Crypto, fetch, History API); allow only vendored/pinned/SHA-checked micro-deps (a ~1KB reactive helper + a tiny Markdown parser). No npm runtime dependency tree. Recorded in [06 — Decided](06-architecture-and-tech-decisions.md#decided) and the Web client section. Remaining sub-choices (which exact micro-libs) deferred to implementation. |
| OQ-3 | ✅ **RESOLVED — No server-side content store; clients are local-first; cross-device sync via bring-your-own storage (BYO), optionally coordinated by a thin server-side E2EE layer** (2026-06-07). | No server-side content database. Each client holds its own store; users connect their own storage backend (WebDAV/S3/Git/cloud-drive) as the sync hub, with client-side encryption. The server may hold opaque E2EE sync-coordination metadata only (OQ-26). See [06 — Decided](06-architecture-and-tech-decisions.md#decided). Spawns OQ-21..OQ-26. |
| OQ-4 | **Client ↔ provider / proxy protocol** — plain REST/JSON (leaning). | The server is only a static host + CORS proxy + OAuth callback, so there's no rich app API to design. |
| OQ-5 | **Real-time updates** now or later? | Proposed: poll/refresh first, push later (FR-SYNC-4). Push is harder with no stateful server. |

## Integrations

| # | Question | Notes |
| --- | --- | --- |
| OQ-6 | **GitHub auth** — fine-grained PAT vs OAuth vs GitHub App? | Affects scopes, setup friction, multi-account. |
| OQ-7 | **Jira auth** — API token (Basic) vs OAuth? Cloud only first? | Cloud first; Server/DC later. |
| OQ-8 | **Polling vs webhooks** for freshness. | Client-side polling is decided; webhooks are impractical without a stateful server. Background sync while apps are closed is limited (accepted trade-off). |
| OQ-8a | **Web CORS** — which provider endpoints work browser-direct vs require the proxy, and is **transient token pass-through** via our proxy acceptable? | Jira likely needs proxying; GitHub may allow some direct calls. Mobile calls direct (no proxy). |
| OQ-9 | **Write-back** to GitHub/Jira (e.g. close/resolve) — in or out? | Currently out of initial scope (FR-INT-11). Confirm. |
| OQ-10 | Should completing an external item in DevTriage be **purely local** or optionally reflect to the source? | Default local-only. |
| OQ-11 | Multi-account per source (e.g. two GitHub orgs / personal + work)? | Affects connection model & data shape. |

## Product behavior

| # | Question | Notes |
| --- | --- | --- |
| OQ-12 | Default **extraction mode** — auto-create vs suggest-and-confirm? | Leaning suggest-and-confirm to avoid noise (FR-EXTRACT-4). |
| OQ-13 | Scope of **action-item detection** — checkboxes only first, or also heuristic sentences? | Checkboxes are MUST; heuristics SHOULD (FR-EXTRACT-5). |
| OQ-14 | Are **Notes vs Ideas** truly distinct entities, or one entity with a flag? | Affects data model ([05](05-data-model.md)). |
| OQ-15 | Any **LLM-assisted** features, and if so where does inference run (local vs hosted)? | Must be opt-in & privacy-preserving (NFR-PRIV-4, FR-EXTRACT-7). |

## Platform & deployment

| # | Question | Notes |
| --- | --- | --- |
| OQ-16 | ✅ **RESOLVED — Multi-tenant hosted service, but no server-side per-user data** (2026-06-07). | Service is for everyone, yet the server holds no customer data; "multi-user" concerns move to the BYO-storage/identity model. See OQ-22. |
| OQ-17 | ✅ **RESOLVED — Hosted by the project owner on a Hetzner VM; not end-user self-hosted** (2026-06-07). | Stateless app host + CORS proxy + OAuth callback. See [06 — Decided](06-architecture-and-tech-decisions.md#decided). |
| OQ-18 | **Minimum OS versions** for iOS/Android targets. | Influences SwiftUI/Compose API availability. |

## BYO storage, sync & identity *(spawned by OQ-3)*

| # | Question | Notes |
| --- | --- | --- |
| OQ-21 | **Which BYO storage backends** to support first? | Candidates: WebDAV, S3-compatible, private Git repo (reuse connected GitHub?), iCloud Drive / Google Drive. Each has different auth & API surface (mind NFR-DEP). |
| OQ-22 | **Identity / auth** — does DevTriage need accounts at all, and how is the **CORS proxy protected from abuse** if there's no login? | With no server data, identity may just be BYO-storage + integration tokens. |
| OQ-23 | **Sync conflict resolution** — CRDT vs last-write-wins vs per-field merge? | Local-first multi-device editing needs a deterministic, no-silent-loss strategy (NFR-REL-3). A server-side opaque version vector (OQ-26) can assist ordering. |
| OQ-24 | **Client-side encryption & key management** for BYO data — passphrase-derived key? device-to-device key exchange? | Storage providers must not be able to read content; keys never reach our server. Server may relay **encrypted** key-exchange envelopes (OQ-26). |
| OQ-25 | **Where do integration tokens live** — device secure store only, or also encrypted in BYO storage so all devices can poll after one connect? | Convenience vs blast radius. |
| OQ-26 | **Server-side E2EE sync-coordination layer** — exactly what does it store, and how is metadata exposure minimized? | Allowed by the owner *if it improves sync*: opaque version pointers, change-notify cursors, encrypted key-exchange envelopes — **never content**. Define the precise schema, retention, and what the server can/can't infer (sizes/timing). |

## Project

| # | Question | Notes |
| --- | --- | --- |
| OQ-19 | **License** for the repository. | README currently says TBD. |
| OQ-20 | Repository structure once coding begins — **monorepo** (backend + web + iOS + android) vs separate repos. | Monorepo likely simplest for a solo/small effort. |

---

> When an item is decided, record the decision in the relevant requirements/architecture
> doc and mark it resolved here (don't delete the history of why).
