# 09 — Open Questions

Decisions still to be made. Each should be resolved before (or early in) the relevant
phase. Owner = project owner unless stated.

## Architecture & stack

| # | Question | Notes / leaning |
| --- | --- | --- |
| OQ-1 | ✅ **RESOLVED — Backend language = Go** (2026-06-07). | Chosen for minimal deps (std-lib HTTP/JSON/crypto/sql), single static binary, and goroutine concurrency for integration polling. Recorded in [06 — Decided](06-architecture-and-tech-decisions.md#decided). One known caveat: SQLite driver (cgo vs pure-Go) deferred to implementation. |
| OQ-2 | ✅ **RESOLVED — Platform-first vanilla TypeScript web client, built with esbuild; build small things in-house, external packages only for big features under a version-aging policy** (2026-06-07). | Lean on native browser APIs (Web Components, IndexedDB, Web Crypto, fetch, History API); reimplement small/utility code (HTTP, reactivity, routing, small parsing) ourselves — no axios-style deps. External packages reserved for genuinely large features (e.g. the rich-text editor), chosen as the latest vuln-free version released ≥1 month ago (NFR-DEP-4/5/6), vendored/pinned/SHA-checked. No npm runtime dependency tree. Recorded in [06 — Decided](06-architecture-and-tech-decisions.md#decided) and the Web client section. |
| OQ-3 | ✅ **RESOLVED — No server-side content store; clients are local-first; cross-device sync via bring-your-own storage (BYO), optionally coordinated by a thin server-side E2EE layer** (2026-06-07). | No server-side content database. Each client holds its own store; users connect their own storage backend (WebDAV/S3/Git/cloud-drive) as the sync hub, with client-side encryption. The server may hold opaque E2EE sync-coordination metadata only (OQ-26). See [06 — Decided](06-architecture-and-tech-decisions.md#decided). Spawns OQ-21..OQ-26. |
| OQ-4 | ✅ **RESOLVED — Plain REST/JSON over HTTPS** (2026-06-07). | The server's only endpoints are the OAuth callback + opaque sync-coordination (REST + WebSocket/SSE for the wake channel); providers expose their own REST APIs; the companion exposes a small local REST endpoint. No rich bespoke app API. |
| OQ-5 | ✅ **RESOLVED — Poll-first; near-real-time content sync via the wake channel; full push later** (2026-06-07). | v1: integrations are **poll + on-demand refresh** (no provider webhooks — a server webhook receiver would expose content/tokens, breaking NFR-PRIV); the companion improves desktop freshness by polling in the background. Content sync between **active** devices is near-real-time via the live wake channel (FR-SYNC-6); background (app-closed) wake is contentless push, phased ([OQ-26a](09-open-questions.md)). Full real-time = FR-SYNC-7 (later). |

## Integrations

| # | Question | Notes |
| --- | --- | --- |
| OQ-6 | ✅ **RESOLVED — GitHub Personal Access Token** (2026-06-07). | Fine-grained PAT with least-privilege read scopes preferred; classic PAT allowed for broad "all repos" reach. Entered by user, stored on-device; no OAuth flow/server callback. github.com first. See [03](03-integration-requirements.md#github-connector--int-gh). |
| OQ-7 | ✅ **RESOLVED — Jira Atlassian API token + email (Basic)** (2026-06-07). | Stored on-device; no OAuth flow. Jira Cloud first; web reaches Jira via the local companion (no browser CORS — [OQ-8a](09-open-questions.md)). See [03](03-integration-requirements.md#jira-connector--int-jira). |
| OQ-8 | ✅ **RESOLVED — Client-side polling, no webhooks** (2026-06-07); see [OQ-5](09-open-questions.md). | Provider webhooks would need a stateful server receiver that sees content/tokens (breaks NFR-PRIV). Background sync while apps are closed is limited — an accepted trade-off; the companion mitigates it on desktop. |
| OQ-8a | ✅ **RESOLVED — Local companion app for web non-CORS providers; no server token proxy** (2026-06-07). | GitHub on web is CORS-direct; Jira/WebDAV on web go through a cross-platform **local companion** on the user's machine (token never touches our server). Companion **required** for those providers on web (no fallback). Mobile always direct. See [06](06-architecture-and-tech-decisions.md). Spawns OQ-29. |
| OQ-9 | ✅ **RESOLVED — Read-only permanently; no write-back** (2026-06-07). | Product decision, not a deferral. Least-privilege read-only token scopes; DevTriage never comments/closes/transitions a source. Status still flows **in** via sync. See [03](03-integration-requirements.md), FR-INT-0c/11. |
| OQ-10 | ✅ **RESOLVED — Completion is local-only** (2026-06-07). | Follows from OQ-9. Completing/dismissing an external item is a local overlay state (`done locally`/`dismissed`) that clears the inbox without touching upstream. See FR-TRIAGE-5, [05](05-data-model.md). |
| OQ-11 | ✅ **RESOLVED — Multiple connections per source, from v1** (2026-06-07). | E.g. work + personal GitHub, multiple Jira sites. Each `ExternalItem` carries its `connection id` and is labeled by account. See FR-INT-1/2/13, INT-COM-1, [05](05-data-model.md). |

## Product behavior

| # | Question | Notes |
| --- | --- | --- |
| OQ-12 | ✅ **RESOLVED — Confidence-based hybrid default** (2026-06-07). | **Auto-create** from explicit checkboxes (high confidence, matches the "auto collected" vision; safe via source-linking + two-way completion); **suggest-and-confirm** for fuzzy heuristic detections (avoids noise). Globally overridable per FR-EXTRACT-4. See [02 — FR-EXTRACT](02-functional-requirements.md#auto-extraction-of-todos--fr-extract). |
| OQ-13 | ✅ **RESOLVED — Phased: explicit markers first, lightweight heuristics later, LLM optional** (2026-06-07). | v1 = GFM checkboxes + inline `TODO:`/`FIXME:`/`@todo` markers (explicit, auto-created). Phase 2 = rule-based in-house heuristics (no heavy NLP/ML dep, NFR-DEP), suggest-and-confirm. Later = optional LLM (FR-EXTRACT-7). See [02 — FR-EXTRACT](02-functional-requirements.md#auto-extraction-of-todos--fr-extract). |
| OQ-14 | ✅ **RESOLVED — One entity with a `kind` flag (`note \| idea`)** (2026-06-07). | Structurally identical (free-form text + tags/project + extraction + search + long-text conflict handling); difference is UX/intent. Shared storage/sync/encryption/search; promotion = flip the flag. See [05 — Data Model](05-data-model.md). |
| OQ-15 | Any **LLM-assisted** features, and if so where does inference run (local vs hosted)? | Must be opt-in & privacy-preserving (NFR-PRIV-4, FR-EXTRACT-7). |

## Platform & deployment

| # | Question | Notes |
| --- | --- | --- |
| OQ-16 | ✅ **RESOLVED — Multi-tenant hosted service, but no server-side per-user data** (2026-06-07). | Service is for everyone, yet the server holds no customer data; "multi-user" concerns move to the BYO-storage/identity model. See OQ-22. |
| OQ-17 | ✅ **RESOLVED — Hosted by the project owner on a Hetzner VM; not end-user self-hosted** (2026-06-07). | Stateless app host + OAuth callback + E2EE sync-coordination (no token proxy). See [06 — Decided](06-architecture-and-tech-decisions.md#decided). |
| OQ-18 | ✅ **RESOLVED — iOS 18+ and Android 14+ (API 34)** (2026-06-07). | Newest-API-only target for a brand-new app: minimal back-compat burden, latest SwiftUI/Compose. Revisit if device-reach data argues otherwise. |

## BYO storage, sync & identity *(spawned by OQ-3)*

| # | Question | Notes |
| --- | --- | --- |
| OQ-21 | ✅ **RESOLVED — Initial BYO backends: WebDAV, Google Drive, Dropbox** (2026-06-07). | All reachable over plain HTTP from mobile + web → in-house adapters (NFR-DEP); content client-side encrypted. **Excluded initially:** private Git repo (no native git on mobile; Contents API rate-limited/clunky; bundled git lib violates NFR-DEP; history model poor for live sync) and iCloud Drive (Apple-only). S3-compatible is a later candidate. Recorded in [06](06-architecture-and-tech-decisions.md). Spawns OQ-27, OQ-28. |
| OQ-27 | **Per-provider auth & verification** — WebDAV (Basic/app-password) vs Google/Dropbox OAuth; can we stay within Google Drive's **app-data folder** / Dropbox **app-folder** scopes to avoid heavy OAuth app verification? | Use **OAuth + PKCE with client-side token exchange** so storage tokens never reach our server (keeps the "no tokens" guarantee absolute); the server only hosts the redirect. WebDAV on web uses the local companion (OQ-8a/28). |
| OQ-28 | ✅ **RESOLVED — WebDAV on web goes through the local companion** (2026-06-07), same as Jira ([OQ-8a](09-open-questions.md)). | Mobile reaches WebDAV directly; Google Drive/Dropbox are CORS-friendly so web reaches them directly. |
| OQ-29 | **Local companion specifics** — port strategy, packaging/signing per OS, pairing-secret UX, and tracking Chrome Private Network Access changes. | Cross-platform Go binary; bind 127.0.0.1; CORS scoped to web origin. |
| OQ-22 | ✅ **RESOLVED — Minimal accounts are required** (2026-06-07). | Needed for a trustworthy user count / active-user metrics (a device heartbeat only counts devices and can't dedupe a person across devices), to gate the sync-coordination service against abuse (NFR-SEC-7), and to anchor per-user sync-coordination. Account = id + auth identity + timestamps + aggregate metrics; **no content/tokens/items**. See [06](06-architecture-and-tech-decisions.md). Spawns OQ-22a. |
| OQ-22a | ✅ **RESOLVED — Passwordless auth: passkey (WebAuthn), Sign in with Google, or Sign in with GitHub** (2026-06-07). | **No direct/password-based account creation;** the account is provisioned automatically on first sign-in. Server stores only a public key (passkey) or provider subject id (Google/GitHub) — no passwords. OAuth uses the Hetzner server's existing callback role. See [06](06-architecture-and-tech-decisions.md). |
| OQ-23 | ✅ **RESOLVED — Pragmatic hybrid** (2026-06-07). | Per-field last-write-wins + add-wins (OR-set) tags + tombstoned deletes, ordered by hybrid logical clocks; long-text conflicts preserved as conflict copies (no silent loss). Built in-house (small CRDTs); merge runs on-device; sequence text-CRDT deferred. Full CRDT rejected (overkill for single-user; cross-platform dep burden, NFR-DEP). See [06](06-architecture-and-tech-decisions.md). |
| OQ-24 | ✅ **RESOLVED — Per-user random DEK (AES‑256‑GCM), platform crypto only; DEK wrapped per-device + by a generated recovery code; new devices enrolled via ECDH+HKDF envelopes relayed opaquely; key decoupled from login** (2026-06-07). | Recovery model = **generated recovery code**. Zero third-party crypto deps (Web Crypto/CryptoKit/JCA), never hand-rolled. Trade-off: losing the recovery code + all devices = unrecoverable (inherent to zero-knowledge). Deferred/optional: passkey PRF unlock, optional passphrase, DEK rotation. See [06](06-architecture-and-tech-decisions.md). |
| OQ-25 | **Where do integration tokens live** — device secure store only, or also encrypted in BYO storage so all devices can poll after one connect? | Convenience vs blast radius. |
| OQ-26 | ✅ **RESOLVED — Opaque per-user schema** (2026-06-07). | Stores only: a sync generation counter + opaque token, a contentless wake/notify channel, a device registry (opaque id + public key + timestamps; labels client-encrypted), and an ephemeral key-exchange envelope mailbox (deleted after pickup). No item-level metadata, no change history; size-padding. Residual metadata (sync timing, device count) minimized and documented. See [06](06-architecture-and-tech-decisions.md). Spawns OQ-26a. |
| OQ-26a | **Mobile background wake via APNs/FCM** — acceptable that Apple/Google see device push tokens + timing (payload contentless)? | The only practical background-wake option; standard. Confirm and document. |

## Project

| # | Question | Notes |
| --- | --- | --- |
| OQ-19 | ✅ **RESOLVED — Proprietary, all rights reserved; repo public for transparency only** (2026-06-07). | Reversed the earlier BSL/source-available leaning: the owner wants **only their own use**. No OSS grant — nobody may use/copy/run/host it without written consent. Repo stays publicly viewable. `LICENSE` added. See README. |
| OQ-30 | ❌ **OBSOLETE** (2026-06-07) — superseded by the proprietary decision in OQ-19; BSL parameters no longer apply. | — |
| OQ-20 | ✅ **RESOLVED — Monorepo** (2026-06-07). | Backend + companion (both Go, **shared connector code**) + web (TS) + iOS (Swift) + Android (Kotlin) + docs in one repo. Solo/small effort → atomic cross-cutting changes, one CI; separate repos only pay off with independent teams/cadences. |

---

> When an item is decided, record the decision in the relevant requirements/architecture
> doc and mark it resolved here (don't delete the history of why).
