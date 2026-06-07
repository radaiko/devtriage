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
| **Accounts / identity** (OQ-22) | **Minimal accounts required** (id + auth identity + timestamps; aggregate metrics). No content/tokens/items. | 2026-06-07 | Trustworthy user count + active-user metrics; gates the CORS proxy against abuse; anchors per-user sync-coordination. |
| **Auth mechanism** (OQ-22a) | **Passwordless: passkey (WebAuthn), Sign in with Google, or Sign in with GitHub.** No password-based signup; account auto-provisioned on first sign-in. | 2026-06-07 | No stored passwords (only public key / provider subject id) → least PII; reuses identities the dev audience already has. |
| **Conflict resolution** (OQ-23) | **Pragmatic hybrid:** per-field LWW + OR-set tags + tombstones, hybrid logical clocks; long-text conflicts kept as conflict copies. Built in-house; on-device merge. | 2026-06-07 | Single-user multi-device → full CRDT is overkill and a cross-platform dep burden (NFR-DEP); hybrid is lossless for the common case and never silently loses data (NFR-REL-3). |
| **Encryption & keys** (OQ-24) | Platform crypto only (zero-dep). Per-user random **DEK**, AES‑256‑GCM content; DEK wrapped per-device and by a **generated recovery code**; new devices enrolled via ECDH+HKDF envelopes relayed opaquely. Key decoupled from login. | 2026-06-07 | True zero-knowledge with passwordless auth; recovery code is the smoothest passwordless recovery; never hand-roll crypto. |
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
  the storage provider can't read it. See the Encryption & key management section below
  (OQ-24).
- **Integration tokens** may also be kept (encrypted) in BYO storage so every device
  can poll after a single connect — *proposed*, see open questions.
- **Sync & conflicts:** resolved via a **pragmatic hybrid** strategy (OQ-23) — see the
  Conflict resolution section below.

## Encryption & key management — **decided** (OQ-24)

All crypto uses **platform primitives** (Web Crypto/SubtleCrypto, Apple CryptoKit,
Android JCA/Keystore) — **never hand-rolled**, and **zero third-party crypto deps**
(NFR-DEP). Everything written to BYO storage *and* every blob the server holds is
ciphertext.

**Key decoupled from login.** Auth is passwordless (passkey/Google/GitHub), and social
logins yield no crypto material — so the encryption key is **independent of the login
method**.

**Key hierarchy:**

- **Content cipher:** **AES‑256‑GCM** (native on all three platforms), fresh nonce per
  record/blob.
- **DEK (Data Encryption Key):** one random per-user symmetric key that encrypts all
  content. Generated on the first device; **never leaves a device in plaintext and never
  reaches our server or any BYO provider.**
- The DEK is **wrapped** two ways:
  1. **Per enrolled device** — protected by a device keypair in the platform secure
     store (Keychain/Keystore; web = non-extractable `CryptoKey`).
  2. **By a generated recovery code** (the chosen recovery model, see below).

**New-device enrollment (zero-knowledge):** the new device generates a keypair, proves
it's logged in, and an existing device wraps the DEK to the new device's public key via
**ECDH (P‑256) + HKDF + AES‑GCM** (all native). The wrapped blob is **relayed through
the server as an opaque encrypted envelope** ([OQ-26](09-open-questions.md)) — the
server sees only ciphertext. If no existing device is available, the user recovers with
the recovery code.

**Recovery model — generated recovery code (OQ-24 decision):** at setup DevTriage
generates a one-time high-entropy **recovery code** the user saves (password manager /
print); it wraps the DEK so the user can recover on a fresh device with no other device
available. It's the smoothest fit with passwordless (nothing to memorize). Honest
trade-off, clearly communicated to the user: **losing the recovery code *and* all
enrolled devices means the data is unrecoverable** — by design, because zero-knowledge
means we *cannot* recover it.

**Deferred / optional later:** passkey **PRF** as a convenience unlock where supported,
an optional user passphrase, and DEK rotation/re-wrap. Tracked under
[OQ-24](09-open-questions.md).

## Conflict resolution — **decided: pragmatic hybrid** (OQ-23)

DevTriage is **single-user across the user's own devices** (no real-time multi-user
collaboration), so full collaborative-CRDT machinery is unnecessary. The bar is
lossless merge for the common case and **no silent data loss** for the rare
concurrent-edit case ([NFR-REL-3](04-non-functional-requirements.md)). A full CRDT
(Automerge/Yjs-class) was rejected: it would need the *same* engine working identically
across TS + Swift + Kotlin — a heavy dependency or error-prone reimplementation
([NFR-DEP](04-non-functional-requirements.md)) — for concurrent-text merging we don't
need.

**The strategy:**

- **Per-record, per-field merge.** Each entity has a stable id; each field carries a
  logical timestamp. Concurrent edits to *different* fields both survive (lossless for
  the common case).
- **Scalar fields → last-write-wins** by logical timestamp.
- **Tags/labels → add-wins set (OR-set)** so concurrent add/remove don't clobber.
- **Deletes → tombstones** (with a retention window) so a delete isn't resurrected by a
  concurrent edit.
- **Long text (note/idea body):** attempt a 3-way/line merge; if it can't merge
  cleanly, **keep both as a conflict copy** flagged for the user — never silently
  discard. May upgrade *just this field* to a text-CRDT later if live collaboration is
  ever wanted.
- **Timestamps = hybrid logical clock** (logical counter + wall-clock), not raw device
  clocks, so clock skew can't distort ordering.
- These structures (LWW-register map, OR-set, tombstones) are small CRDTs **built
  in-house** — no heavy dependency. The only "too big to build" piece (a sequence
  text-CRDT) is deferred.
- **Server role:** the opaque version vector ([OQ-26](09-open-questions.md)) only helps
  detect/order changes; the **merge runs on-device** (the server can't read content).

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

## Auth / identity — **decided: minimal accounts required** (OQ-22)

DevTriage **requires a user account / login**. Accounts give the operator a
trustworthy **user count** and active-user metrics (an anonymous device heartbeat would
only count devices, reset on reinstall, and can't dedupe a person across web+iOS+Android),
and they double as the **gate that stops the CORS proxy being an open relay**
([NFR-SEC-7](04-non-functional-requirements.md)) and the **per-user anchor for the E2EE
sync-coordination metadata** ([OQ-26](09-open-questions.md)).

**Data minimization — this does not break the privacy promise.** An account is the
smallest possible record: a user id, an auth identity, and timestamps (e.g. created /
last-seen), plus **aggregate** usage metrics. The server still stores **no notes,
todos, documents, integration tokens, or fetched items** — those stay on-device and in
BYO storage.

**Transparency (FR-SET-0a).** Because the product promise is "we don't hold your
data," the signup/login flow must **tell the user why an account is needed** — proxy
abuse protection, E2EE sync coordination, and user-count metrics — and reassure that no
content, tokens, or fetched items are stored server-side. Don't make the account feel
like an unexplained data grab.

**Auth mechanism — decided (OQ-22a): passwordless only.** Three sign-in options:

- **Passkey (WebAuthn)** — server stores only a public key / credential id; can be
  email-less (least PII).
- **Sign in with Google** and **Sign in with GitHub** — server stores only the provider
  subject id (+ email only if needed); OAuth runs through the server's existing callback
  role.

**No direct/password-based account creation** — there is no email+password signup; the
account is **provisioned automatically on first sign-in**. The server never stores
passwords (NFR-SEC). Note: "Sign in with GitHub" (a server-side identity assertion) is
**separate** from the client-side GitHub *integration* token used for polling — they
are different credentials with different trust boundaries.

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
