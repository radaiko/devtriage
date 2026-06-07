# 06 — Architecture & Tech Decisions

> **Status:** core shape now **decided** (see the Decided table). Remaining details are
> tracked in [09 — Open Questions](09-open-questions.md). Items marked *proposed* are
> still open.

## Firm constraints

1. **Native mobile clients** — iOS in **Swift/SwiftUI**, Android in
   **Kotlin/Jetpack Compose**. No cross-platform JS runtimes (React Native, Flutter).
2. **Minimal dependency surface** — avoid large third-party dependency trees,
   **especially npm**. See [NFR-DEP](04-non-functional-requirements.md#dependency--supply-chain-policy-nfr-dep).
3. **The server stores no customer content.** DevTriage is operated as a hosted
   service, but the server never persists documents, notes, todos, integration tokens,
   or fetched items. It **may** store only **E2EE / opaque sync-coordination metadata**
   (never readable content) where that improves the sync workflow.
   See [NFR-PRIV](04-non-functional-requirements.md#privacy--data-ownership-nfr-priv).

## Decided

| Decision | Choice | Date | Rationale (summary) |
| --- | --- | --- | --- |
| **Backend language** (OQ-1) | **Go** | 2026-06-07 | Std-lib covers HTTP server+client, JSON, crypto; single static binary; good concurrency. |
| **Hosting model** (OQ-17) | **Hosted multi-tenant service on a Hetzner VM**, operated by the project owner. **Not** end-user self-hosted. | 2026-06-07 | Owner wants one service for everyone, without the burden of self-hosting. |
| **Server data** (OQ-3, OQ-16) | **No customer content** — no documents/notes/todos, no tokens, no fetched items. **May** hold **E2EE/opaque sync-coordination metadata** only. | 2026-06-07 | Minimize liability/privacy exposure; allow a thin coordination layer if it improves sync, as long as content stays unreadable to the server. |
| **Client architecture** | **Local-first.** Each client holds its own data and is the working source of truth. | 2026-06-07 | Required once the server holds no content. |
| **Cross-device sync** | **Bring-your-own storage (BYO)** for content, **optionally coordinated** by a thin server-side E2EE sync service. | 2026-06-07 | Content travels through the user's own storage; the server may coordinate (change-notify / version / key-exchange) without reading content. |
| **Initial BYO backends** (OQ-21) | **WebDAV, Google Drive, Dropbox.** | 2026-06-07 | All HTTP-reachable from mobile+web → in-house adapters. Git-repo (mobile/git issues, NFR-DEP) and iCloud (Apple-only) excluded initially. |
| **Integration collection** | **Client-side polling.** Apps call GitHub/Jira directly; tokens live on-device. | 2026-06-07 | Server never sees tokens or fetched items. |
| **Web client** (OQ-2) | **Platform-first** (Web Components, IndexedDB, Web Crypto, fetch) **TypeScript** app built with **esbuild**. Small/utility code is **built in-house** (no axios-style deps); external packages **only for big features** (e.g. the text editor), under the version-aging policy. No npm runtime tree. | 2026-06-07 | Local-first rules out server-rendered; lean on the platform + build small things ourselves; reserve deps for what's too big to reimplement (NFR-DEP-4/5/6). |

## High-level shape

```
        ┌──────────────────────────────────────────────────────────┐
        │               Hetzner VM — thin Go service                 │
        │  • serves the web client    • OAuth callback handling      │
        │  • thin CORS proxy for web → GitHub/Jira (see notes)       │
        │  • E2EE sync coordination (opaque version/notify/keys)     │
        │  • holds NO content, NO tokens, NO fetched items           │
        └───────▲───────────────────▲───────────────────▲───────────┘
                │ static app / proxy │                   │
        ┌───────┴──────┐    ┌────────┴─────┐    ┌────────┴─────┐
        │  Web client  │    │ iOS (Swift)  │    │ Android (Kt) │   local-first
        │ local store  │    │ local store  │    │ local store  │   (source of truth)
        └──┬────────┬──┘    └──┬────────┬──┘    └──┬────────┬──┘
           │        │          │        │          │        │
   direct  │        │  sync     │       │  sync     │        │ direct API calls
   API     │        ▼           ▼       ▼           ▼        │ (mobile: no proxy)
   (mobile)│   ┌─────────────────────────────────────┐      │
           │   │   User's OWN storage (BYO sync hub)  │      │
           │   │  WebDAV / S3 / private Git / etc.    │      │
           │   │   (encrypted by the client)         │      │
           │   └─────────────────────────────────────┘      │
           └───────────────► GitHub / Jira ◄────────────────┘
```

Two independent data flows, **neither of which touches our server's storage**:

1. **Sync flow** — clients read/write the user's own data (todos, notes, ideas,
   overlays) to the **user's BYO storage**, encrypted client-side.
2. **Integration flow** — clients poll **GitHub/Jira directly** with on-device tokens
   (web routes through the stateless CORS proxy where the provider requires it).

## The server (Hetzner VM) — what it is and isn't

**Is:** a small Go service that (a) serves the web client, (b) handles OAuth
redirect/callback for connecting GitHub/Jira, (c) provides a thin CORS proxy so the
**web** client can reach provider APIs that don't allow browser-origin calls, and (d)
optionally runs a **thin E2EE sync-coordination layer** (see below).

**Is not:** a content store or a token vault. It persists **no documents, notes,
todos, tokens, or fetched items** — only opaque sync-coordination metadata, if any. A
wiped VM loses no user content; everything readable lives on devices and in the user's
BYO storage.

> **Why Go is still the right pick:** even as a thin stateless host/proxy, Go's std-lib
> HTTP server + single static binary + easy deployment on a single VM fit perfectly,
> with essentially zero third-party dependencies (NFR-DEP).

## Local-first clients

Each client (web, iOS, Android) keeps a **local store** that is the working source of
truth:

- **iOS:** local store (e.g. SQLite/GRDB or SwiftData); tokens in **Keychain**.
- **Android:** local store (e.g. SQLite/Room); tokens in **Keystore**.
- **Web:** browser local storage (**IndexedDB**); see the dedicated web-client section
  below for the dependency approach.

Clients work fully offline against the local store and reconcile via BYO storage when
connectivity returns (NFR-OFF).

## Web client — **decided: platform-first, vendored micro-deps** (OQ-2)

The local-first decision makes the web client a real client-side app, so a
server-rendered approach is off the table. To honor
[NFR-DEP](04-non-functional-requirements.md#dependency--supply-chain-policy-nfr-dep)
we **lean on the browser platform** and add only tiny, vendored, zero-transitive
dependencies — **no `node_modules` runtime tree, no silent updates.**

**Default = build it ourselves.** Small/utility functionality is reimplemented in-house
on native APIs rather than pulled as a package — we can make it leaner and more
performant and carry zero supply-chain risk (e.g. use native `fetch`, **not** axios;
hand-roll reactivity, routing, small parsing). This is [NFR-DEP-4](04-non-functional-requirements.md#dependency--supply-chain-policy-nfr-dep).

**Exception = genuinely large features.** Where reimplementing is too much effort —
most clearly a **rich text / Markdown editor** — we use an external package
([NFR-DEP-5](04-non-functional-requirements.md#dependency--supply-chain-policy-nfr-dep)).
When we do, we follow the **version-aging policy**: the latest version with **no known
vulnerabilities** that was **released at least a month ago**
([NFR-DEP-6](04-non-functional-requirements.md#dependency--supply-chain-policy-nfr-dep)),
vendored/pinned/SHA-checked.

**Platform-first mapping (zero dependencies):**

| Need | Native browser API |
| --- | --- |
| Local store (local-first) | **IndexedDB** |
| E2EE for BYO sync | **Web Crypto (SubtleCrypto)** — never hand-roll crypto |
| Poll GitHub/Jira | **fetch** |
| UI components | **Web Components** (Custom Elements + `<template>`) |
| Routing | **History API / URLPattern** |
| Reactivity | small **hand-rolled** store/signals |
| HTTP | native **fetch** (no axios-style client) |
| Markdown parse (rendering + checkbox detection) | **hand-rolled** subset (needed for FR-EXTRACT anyway) |

**In-house by default:** reactivity, routing, HTTP, and small parsing are built on the
native APIs above — no packages.

**External package expected for one big feature:** the **rich text / Markdown editor**
is the realistic case where reimplementing is not worth it. Selected per the
version-aging policy (NFR-DEP-5/6) and vendored. Any other external dependency must
clear the same "too big to build ourselves" bar.

**Build tooling:** author in **TypeScript**, bundle/transpile with **esbuild** — a
single native Go binary with no dependency tree of its own (symmetry with the Go
backend). Fallback if even esbuild is unwanted: `tsc`-only emit of ES modules loaded
natively, or plain-JS no-build.

**Vendoring policy (applies repo-wide):** any third-party file is committed under a
`vendor/`-style path, pinned to an **exact** version with a recorded checksum, reviewed
on intake, and **updated only by a deliberate, reviewed manual step** — never via
background `npm install` or an auto-update bot. Reinforces
[NFR-DEP-3](04-non-functional-requirements.md#dependency--supply-chain-policy-nfr-dep)
and [NFR-DEP-10](04-non-functional-requirements.md#dependency--supply-chain-policy-nfr-dep).

## Bring-your-own storage (BYO sync)

- A **storage adapter** abstracts the user's chosen backend behind one interface
  (`read`, `write`, `list`, `delete` of encrypted objects).
- **Initial backends (decided — OQ-21): WebDAV, Google Drive, Dropbox.** All three are
  reachable over plain HTTP from both mobile and web, so adapters are built in-house
  (no heavy SDKs, per NFR-DEP). Use scoped access where available (Google Drive
  **app-data folder**, Dropbox **app-folder**) to minimize permissions and OAuth
  verification burden.
- **Deliberately excluded from the initial set** (kept possible later via the adapter
  interface): a **private Git repo** (no native git on mobile; the Contents API is
  rate-limited and clunky for frequent small writes, and a bundled git library violates
  NFR-DEP; git's history model is a poor fit for live sync) and **iCloud Drive**
  (Apple-only, so it can't serve Android/web). S3-compatible is a plausible later add.
- **Client-side encryption:** data written to BYO storage is encrypted by the client so
  the storage provider can't read it. Key management is an [open question](09-open-questions.md).
- **Integration tokens** may also be kept (encrypted) in BYO storage so every device
  can poll after a single connect — *proposed*, see open questions.
- **Sync & conflicts:** local-first multi-device editing needs a conflict-resolution
  strategy (CRDT vs last-write-wins vs per-field). [Open](09-open-questions.md).

## Sync coordination (server-side, E2EE — recommended)

BYO storage alone works, but pure poll-the-bucket sync is slow and makes conflict
ordering and new-device onboarding harder. A **thin, opaque coordination layer** on the
Hetzner server materially improves the workflow **without the server reading any
content**. It may hold, per user:

- an **opaque version pointer / vector** so devices detect "is there anything new?"
  cheaply and order changes (helps conflict resolution, [OQ-23](09-open-questions.md));
- **change-notification cursors / wake signals** so a write on one device promptly
  nudges the others (enables near-real-time sync — the practical form of `FR-SYNC-6`);
- **encrypted key-exchange envelopes** to bootstrap a new device into the user's
  encryption keys ([OQ-24](09-open-questions.md)).

Constraints on this layer:

- It stores **only ciphertext / opaque counters** — never documents, notes, todos,
  tokens, or fetched items.
- The actual content still flows through **BYO storage**; coordination metadata never
  substitutes for it.
- Minimize even *metadata* exposure (sizes, timing) where practical, and make it clear
  what the server can and cannot infer ([OQ-26](09-open-questions.md)).

## Integration collection (client-side)

- Connectors run **inside each client**; the user's tokens never leave the device's
  secure store (mobile) / browser (web).
- **Mobile** apps call GitHub/Jira **directly**.
- **Web** must contend with browser CORS: some provider endpoints allow browser-origin
  calls, others (notably Jira) do not. Where needed, the web client routes through the
  Hetzner **CORS proxy**, which **forwards but does not store** the request/token.
  Whether transient token pass-through via the proxy is acceptable — and which
  providers work direct vs proxied — is an [open question](09-open-questions.md).
- Because polling is client-side, **background sync while the app is closed is limited**
  (esp. web). This is an accepted trade-off of the "server stores nothing" decision.

See [03 — Integration Requirements](03-integration-requirements.md) for item-level
detail.

## Auth / identity

With no server-side storage, DevTriage may need **no accounts of its own**: a user's
"identity" is their connected integrations plus their BYO storage. The open items are
whether any login is needed at all and how to protect the CORS proxy from abuse — see
[09 — Open Questions](09-open-questions.md).

## Security posture (summary)

- Tokens live only on-device (Keychain/Keystore/browser) and, if synced, only as
  **client-encrypted** data in the user's BYO storage (NFR-SEC-1/3).
- The server never stores tokens or content; the web CORS proxy handles them only in
  transit, over TLS (NFR-SEC-2/4).
- Every dependency pinned and integrity-checked (NFR-DEP-3); CI dependency scanning
  once code begins (NFR-DEP-8).

## What this buys us against the constraints

- **No customer data on the server** — minimal liability/privacy exposure.
- **Native mobile** as required; **npm avoided** on the backend entirely.
- **User owns their data** — it lives on their devices and in storage they control.

> Open architectural decisions are consolidated in
> [09 — Open Questions](09-open-questions.md).
