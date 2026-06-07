# 09 — Open Questions

Decisions still to be made. Each should be resolved before (or early in) the relevant
phase. Owner = project owner unless stated.

## Architecture & stack

| # | Question | Notes / leaning |
| --- | --- | --- |
| OQ-1 | ✅ **RESOLVED — Backend language = Go** (2026-06-07). | Chosen for minimal deps (std-lib HTTP/JSON/crypto/sql), single static binary, and goroutine concurrency for integration polling. Recorded in [06 — Decided](06-architecture-and-tech-decisions.md#decided). One known caveat: SQLite driver (cgo vs pure-Go) deferred to implementation. |
| OQ-2 | ✅ **RESOLVED — Platform-first vanilla TypeScript web client, built with esbuild; build small things in-house, external packages only for big features under a version-aging policy** (2026-06-07). | Lean on native browser APIs (Web Components, IndexedDB, Web Crypto, fetch, History API); reimplement small/utility code (HTTP, reactivity, routing, small parsing) ourselves — no axios-style deps. External packages reserved for genuinely large features (e.g. the rich-text editor), chosen as the latest vuln-free version released ≥1 month ago (NFR-DEP-4/5/6), vendored/pinned/SHA-checked. No npm runtime dependency tree. Recorded in [06 — Decided](06-architecture-and-tech-decisions.md#decided) and the Web client section. |
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
| OQ-21 | ✅ **RESOLVED — Initial BYO backends: WebDAV, Google Drive, Dropbox** (2026-06-07). | All reachable over plain HTTP from mobile + web → in-house adapters (NFR-DEP); content client-side encrypted. **Excluded initially:** private Git repo (no native git on mobile; Contents API rate-limited/clunky; bundled git lib violates NFR-DEP; history model poor for live sync) and iCloud Drive (Apple-only). S3-compatible is a later candidate. Recorded in [06](06-architecture-and-tech-decisions.md). Spawns OQ-27, OQ-28. |
| OQ-27 | **Per-provider auth & verification** — WebDAV (Basic/app-password) vs Google/Dropbox OAuth; can we stay within Google Drive's **app-data folder** / Dropbox **app-folder** scopes to avoid heavy OAuth app verification? | OAuth callbacks run through the Hetzner server's existing callback role. |
| OQ-28 | **WebDAV web CORS** — most WebDAV servers don't send CORS headers, so the web client likely needs the Hetzner proxy (ties to [OQ-8a](09-open-questions.md)); confirm scope. | Mobile reaches WebDAV directly. |
| OQ-22 | ✅ **RESOLVED — Minimal accounts are required** (2026-06-07). | Needed for a trustworthy user count / active-user metrics (a device heartbeat only counts devices and can't dedupe a person across devices), to gate the CORS proxy against abuse (NFR-SEC-7), and to anchor per-user sync-coordination. Account = id + auth identity + timestamps + aggregate metrics; **no content/tokens/items**. See [06](06-architecture-and-tech-decisions.md). Spawns OQ-22a. |
| OQ-22a | ✅ **RESOLVED — Passwordless auth: passkey (WebAuthn), Sign in with Google, or Sign in with GitHub** (2026-06-07). | **No direct/password-based account creation;** the account is provisioned automatically on first sign-in. Server stores only a public key (passkey) or provider subject id (Google/GitHub) — no passwords. OAuth uses the Hetzner server's existing callback role. See [06](06-architecture-and-tech-decisions.md). |
| OQ-23 | ✅ **RESOLVED — Pragmatic hybrid** (2026-06-07). | Per-field last-write-wins + add-wins (OR-set) tags + tombstoned deletes, ordered by hybrid logical clocks; long-text conflicts preserved as conflict copies (no silent loss). Built in-house (small CRDTs); merge runs on-device; sequence text-CRDT deferred. Full CRDT rejected (overkill for single-user; cross-platform dep burden, NFR-DEP). See [06](06-architecture-and-tech-decisions.md). |
| OQ-24 | ✅ **RESOLVED — Per-user random DEK (AES‑256‑GCM), platform crypto only; DEK wrapped per-device + by a generated recovery code; new devices enrolled via ECDH+HKDF envelopes relayed opaquely; key decoupled from login** (2026-06-07). | Recovery model = **generated recovery code**. Zero third-party crypto deps (Web Crypto/CryptoKit/JCA), never hand-rolled. Trade-off: losing the recovery code + all devices = unrecoverable (inherent to zero-knowledge). Deferred/optional: passkey PRF unlock, optional passphrase, DEK rotation. See [06](06-architecture-and-tech-decisions.md). |
| OQ-25 | **Where do integration tokens live** — device secure store only, or also encrypted in BYO storage so all devices can poll after one connect? | Convenience vs blast radius. |
| OQ-26 | ✅ **RESOLVED — Opaque per-user schema** (2026-06-07). | Stores only: a sync generation counter + opaque token, a contentless wake/notify channel, a device registry (opaque id + public key + timestamps; labels client-encrypted), and an ephemeral key-exchange envelope mailbox (deleted after pickup). No item-level metadata, no change history; size-padding. Residual metadata (sync timing, device count) minimized and documented. See [06](06-architecture-and-tech-decisions.md). Spawns OQ-26a. |
| OQ-26a | **Mobile background wake via APNs/FCM** — acceptable that Apple/Google see device push tokens + timing (payload contentless)? | The only practical background-wake option; standard. Confirm and document. |

## Project

| # | Question | Notes |
| --- | --- | --- |
| OQ-19 | **License** for the repository. | README currently says TBD. |
| OQ-20 | Repository structure once coding begins — **monorepo** (backend + web + iOS + android) vs separate repos. | Monorepo likely simplest for a solo/small effort. |

---

> When an item is decided, record the decision in the relevant requirements/architecture
> doc and mark it resolved here (don't delete the history of why).
