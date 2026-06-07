# 06 — Architecture & Tech Decisions

> **Status:** core shape now **decided** (see the Decided table). Remaining details are
> tracked in [09 — Open Questions](09-open-questions.md). Items marked *proposed* are
> still open.

## Firm constraints

1. **Native mobile clients** — iOS in **Swift/SwiftUI** (**iOS 18+**), Android in
   **Kotlin/Jetpack Compose** (**Android 14+ / API 34**) (OQ-18). No cross-platform JS
   runtimes (React Native, Flutter).
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
| **Accounts / identity** (OQ-22) | **Minimal accounts required** (id + auth identity + timestamps; aggregate metrics). No content/tokens/items. | 2026-06-07 | Trustworthy user count + active-user metrics; gates the sync-coordination service against abuse; anchors per-user sync-coordination. |
| **Auth mechanism** (OQ-22a) | **Passwordless: passkey (WebAuthn), Sign in with Google, or Sign in with GitHub.** No password-based signup; account auto-provisioned on first sign-in. | 2026-06-07 | No stored passwords (only public key / provider subject id) → least PII; reuses identities the dev audience already has. |
| **Conflict resolution** (OQ-23) | **Pragmatic hybrid:** per-field LWW + OR-set tags + tombstones, hybrid logical clocks; long-text conflicts kept as conflict copies. Built in-house; on-device merge. | 2026-06-07 | Single-user multi-device → full CRDT is overkill and a cross-platform dep burden (NFR-DEP); hybrid is lossless for the common case and never silently loses data (NFR-REL-3). |
| **Encryption & keys** (OQ-24) | Platform crypto only (zero-dep). Per-user random **DEK**, AES‑256‑GCM content; DEK wrapped per-device and by a **generated recovery code**; new devices enrolled via ECDH+HKDF envelopes relayed opaquely. Key decoupled from login. | 2026-06-07 | True zero-knowledge with passwordless auth; recovery code is the smoothest passwordless recovery; never hand-roll crypto. |
| **Sync coordination schema** (OQ-26) | Per-user, opaque only: generation counter + token, contentless wake channel, device registry (public keys), ephemeral key-exchange envelope mailbox. No item-level metadata; no history. | 2026-06-07 | Speeds sync and enables device enrollment while keeping the server zero-knowledge; residual metadata (timing, device count) minimized and documented. |
| **GitHub integration auth** (OQ-6) | **Personal Access Token** (fine-grained preferred; classic allowed for broad reach). | 2026-06-07 | Simplest for client-side polling — no OAuth flow/server callback; token stays on-device. |
| **Jira integration auth** (OQ-7) | **Atlassian API token + email (Basic)**, Jira Cloud first. | 2026-06-07 | Same rationale; web reaches Jira via the local companion (Jira lacks browser CORS). |
| **Web non-CORS access** (OQ-8a) | **Local companion app** (cross-platform, Go) for web + Jira/WebDAV; **no server token proxy**. Required for those providers on web (no fallback). | 2026-06-07 | Keeps tokens entirely off our server; GitHub stays CORS-direct; bonus desktop background sync. |
| **Integration collection** | **Client-side polling.** Apps call GitHub/Jira directly; tokens live on-device. | 2026-06-07 | Server never sees tokens or fetched items. |
| **Web client** (OQ-2) | **Platform-first** (Web Components, IndexedDB, Web Crypto, fetch) **TypeScript** app built with **esbuild**. Small/utility code is **built in-house** (no axios-style deps); external packages **only for big features** (e.g. the text editor), under the version-aging policy. No npm runtime tree. | 2026-06-07 | Local-first rules out server-rendered; lean on the platform + build small things ourselves; reserve deps for what's too big to reimplement (NFR-DEP-4/5/6). |
| **Integration model** (OQ-9/10/11) | **Read-only, permanently** (least-privilege read scopes; never modifies sources); completion is **local-only**; **multiple connections per source** from v1. | 2026-06-07 | Triage layer; source stays source of truth; smaller token blast radius; work+personal accounts are common. |
| **Update freshness** (OQ-5) | **Poll-first** integrations (no provider webhooks); near-real-time content sync between active devices via the wake channel; background push contentless & phased. | 2026-06-07 | Webhooks would expose content/tokens to the server (NFR-PRIV); the wake channel already gives live sync. |
| **Mobile OS targets** (OQ-18) | **iOS 18+**, **Android 14+ (API 34)**. | 2026-06-07 | Brand-new app → newest-API-only minimizes back-compat burden; revisit on device-reach data. |
| **License** (OQ-19) | **Source-available — Business Source License (BSL 1.1)**; parameters TBD (OQ-30). | 2026-06-07 | Readable/self-hostable but bars a competing hosted service → keeps the hosted business defensible. |

## High-level shape

```
        ┌──────────────────────────────────────────────────────────┐
        │               Hetzner VM — thin Go service                 │
        │  • serves the web client    • OAuth callback handling      │
        │  • E2EE sync coordination (opaque version/notify/keys)     │
        │  • holds NO content, NO tokens, NO fetched items           │
        │  • NEVER proxies provider tokens                           │
        └───────▲───────────────────▲───────────────────▲───────────┘
                │ static app / coord │                   │
        ┌───────┴──────┐    ┌────────┴─────┐    ┌────────┴─────┐
        │  Web client  │    │ iOS (Swift)  │    │ Android (Kt) │   local-first
        │ local store  │    │ local store  │    │ local store  │   (source of truth)
        └──┬───────┬───┘    └──┬────────┬──┘    └──┬────────┬──┘
           │       │           │        │          │        │
   via     │       │   sync     │       │  sync     │        │ direct API calls
 companion │       ▼            ▼       ▼           ▼        │ (mobile: always direct)
 (Jira/    │  ┌─────────────────────────────────────┐      │
  WebDAV)  │  │   User's OWN storage (BYO sync hub)  │      │
   +direct │  │  WebDAV / Google Drive / Dropbox     │      │
  (GitHub) │  │   (encrypted by the client)         │      │
           │  └─────────────────────────────────────┘      │
           ▼                                                │
  ┌─────────────────────┐                                  │
  │  Local companion app │ ──► GitHub / Jira / WebDAV       │
  │  (user's machine,    │ ◄───────────────────────────────┘
  │   holds the tokens)  │
  └─────────────────────┘
```

Data flows, **none of which let our server touch content or tokens**:

1. **Sync flow** — clients read/write the user's own data to the **user's BYO storage**,
   encrypted client-side.
2. **Integration flow** — clients poll providers with on-device tokens. **Mobile** calls
   directly. **Web** calls GitHub directly (CORS-friendly); for **Jira / WebDAV** (no
   browser CORS) the web client calls a **local companion app** on the user's own
   machine, which holds the token and forwards the request. The server is never in this
   path.
3. **Coordination flow** — opaque E2EE sync metadata only (version/notify/key-exchange).

## The server (Hetzner VM) — what it is and isn't

**Is:** a small Go service that (a) serves the web client, (b) handles OAuth
redirect/callback for login and BYO cloud-storage connect, and (c) runs the **thin E2EE
sync-coordination layer** (see below).

**Is not:** a content store, a token vault, **or a provider proxy**. It persists **no
documents, notes, todos, tokens, or fetched items** — only opaque sync-coordination
metadata. It **never** relays provider/storage tokens (the local companion handles the
web non-CORS case). A wiped VM loses no user content.

> **Why Go is still the right pick:** even as a thin stateless host, Go's std-lib HTTP
> server + single static binary + easy deployment on a single VM fit perfectly, with
> essentially zero third-party dependencies (NFR-DEP). Go is **also ideal for the local
> companion** — a cross-platform single binary that can reuse the connector code.

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

## Sync coordination — **decided: opaque schema** (OQ-26)

BYO storage alone works, but pure poll-the-bucket sync is slow and makes conflict
ordering and device onboarding harder. A **thin, opaque coordination layer** on the
Hetzner server materially improves the workflow **without the server reading any
content**. It is per-user (anchored to the account, OQ-22) and holds **only** public
keys, opaque counters, and ciphertext.

**Schema (per user):**

| Item | Purpose | Privacy form |
| --- | --- | --- |
| **Sync generation counter** + opaque token | Cheap "is there anything new / what's the latest" so devices know when to pull from BYO (vs blind polling) | Monotonic integer + opaque ciphertext token. Per-item version vectors stay in the **encrypted BYO payload**, never here — so the server can't learn item counts/ids |
| **Wake / notify channel** | A write on one device nudges the others → near-real-time sync (`FR-SYNC-6`) | Contentless "gen bumped" signal over WebSocket/SSE |
| **Device registry** | Lets an enrolled device wrap the DEK to a new device's public key (OQ-24); supports revocation | Opaque device id + **public key** (not secret) + timestamps; any human label is **client-encrypted** |
| **Key-exchange envelope mailbox** | Delivers the wrapped DEK during enrollment (OQ-24) | Ephemeral ciphertext, **deleted after pickup** |

**Never stored here:** documents/notes/todos, the DEK or any plaintext key, integration
tokens, fetched items, readable device descriptions, or readable version-vector
contents.

**Minimization rules:**

- Keep **only** counters, public keys, opaque tokens, ephemeral ciphertext, coarse
  timestamps. No item-level metadata (no item ids/counts) — only the aggregate
  generation token.
- Ephemeral data (envelopes, wake signals) is deleted after delivery; **no change
  history** is retained.
- Pad envelope/blob sizes to buckets to limit size-based inference.
- **Residual metadata acknowledged:** the server can still infer *sync timing/frequency*
  and *device count*. Minimize and document this; it's the cost of coordination.

**Transport:** TLS; live wake via WebSocket/SSE with REST poll as fallback. Mobile
**background** wake needs APNs/FCM (Apple/Google see device push tokens + timing); the
payload stays **contentless** — tracked as [OQ-26a](09-open-questions.md).

## Integration collection (client-side)

- Connectors run **inside each client**; the user's tokens never leave the device's
  secure store (mobile) / the browser or local companion (web).
- **Mobile** apps call GitHub/Jira **directly** (no CORS restriction).
- **Web** contends with browser CORS: **GitHub** is CORS-friendly → called **directly**;
  **Jira and WebDAV** are not → reached via the **local companion app** (OQ-8a), never
  through our server.
- Because polling is client-side, **background sync while the app is closed is limited**
  on web — *except* where the local companion runs (it can keep polling and wake the web
  app). Mobile background wake uses contentless push ([OQ-26a](09-open-questions.md)).

## Local companion app — **decided** (OQ-8a)

A small **cross-platform local app** (Go — single binary, minimal deps, can reuse the
connector code) that runs on the user's own machine and lets the **web** client reach
providers that browsers can't call directly (**Jira, WebDAV**). The web app calls
`http://127.0.0.1:PORT`; the companion holds the token locally and forwards the request.
**The token never touches our server** — keeping "the server sees no tokens" absolute.

- **Required for web + Jira/WebDAV; no server fallback.** If the companion isn't
  installed, the web app prompts to install it (or use mobile). GitHub-only web users
  don't need it (GitHub is CORS-direct).
- **Browser → localhost mechanics:** the companion returns CORS headers scoped to the
  DevTriage web origin; `http://localhost` is a browser "secure context" so HTTPS→localhost
  isn't blocked as mixed content; it handles Chrome **Private Network Access** preflight
  (`Access-Control-Allow-Private-Network`).
- **Abuse protection:** bind to `127.0.0.1` only, CORS origin allowlist, **plus a
  pairing secret** so only the real DevTriage web app can use it.
- **Bonus:** because it runs locally, the companion can poll Jira/GitHub even when the
  browser tab is closed and then wake the web app — desktop background sync.
- Open specifics (port strategy, distribution/signing, PNA evolution) tracked as
  [OQ-29](09-open-questions.md).

See [03 — Integration Requirements](03-integration-requirements.md) for item-level
detail.

## Auth / identity — **decided: minimal accounts required** (OQ-22)

DevTriage **requires a user account / login**. Accounts give the operator a
trustworthy **user count** and active-user metrics (an anonymous device heartbeat would
only count devices, reset on reinstall, and can't dedupe a person across web+iOS+Android),
and they double as the **gate that protects the sync-coordination service from abuse**
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

- Tokens live only on-device (Keychain/Keystore/browser/local companion) and, if synced,
  only as **client-encrypted** data in the user's BYO storage (NFR-SEC-1/3).
- The server **never** stores or relays tokens; the web non-CORS case is handled by the
  user's local companion, not our server (NFR-SEC-4). All traffic over TLS (NFR-SEC-2).
- Every dependency pinned and integrity-checked (NFR-DEP-3); CI dependency scanning
  once code begins (NFR-DEP-8).

## What this buys us against the constraints

- **No customer data on the server** — minimal liability/privacy exposure.
- **Native mobile** as required; **npm avoided** on the backend entirely.
- **User owns their data** — it lives on their devices and in storage they control.

> Open architectural decisions are consolidated in
> [09 — Open Questions](09-open-questions.md).
