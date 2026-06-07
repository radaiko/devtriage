# 04 — Non-Functional Requirements

Legend: 🔴 MUST · 🟠 SHOULD · 🟢 MAY

## Dependency & supply-chain policy — `NFR-DEP`

> This is a **first-class, explicit constraint** from the project owner: recent
> ecosystem supply-chain attacks (especially in npm) make dependency hygiene a core
> requirement, not an afterthought.

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-DEP-1 | 🔴 | Minimize the number of third-party runtime dependencies; prefer standard libraries. |
| NFR-DEP-2 | 🔴 | Avoid large, deeply-transitive dependency trees — **the npm ecosystem in particular is to be avoided** for production code. |
| NFR-DEP-3 | 🔴 | Every third-party dependency must be deliberately chosen and **pinned to an exact version** (no semver ranges / `^` / `~` / floating tags) and integrity-checked (lockfile / checksums / vendoring). |
| NFR-DEP-4 | 🔴 | **Reimplement small/utility functionality in-house** using native/standard APIs rather than adding a dependency (e.g. use native `fetch`, **not** an HTTP client like axios; hand-roll reactivity, routing, small parsing). We can keep these leaner, more performant, and dependency-free. |
| NFR-DEP-5 | 🔴 | **External packages are reserved for genuinely large features** that are impractical to reimplement (e.g. a rich text / Markdown editor). The bar is "too much effort to build and maintain ourselves," not "convenient." |
| NFR-DEP-6 | 🔴 | **Dependency intake / version-aging policy:** when a dependency is justified, adopt the **latest version that (a) has no known vulnerabilities and (b) was released at least one month ago.** The cooldown window guards against freshly-published compromised releases. |
| NFR-DEP-10 | 🔴 | **Updates are manual and deliberate only.** No automated dependency updates or auto-merging update bots (e.g. Dependabot/Renovate auto-merge). Every version bump is human-initiated, re-passes the NFR-DEP-6 checks, is reviewed, and re-pinned/re-checksummed. |
| NFR-DEP-7 | 🟠 | Dependencies should be auditable; prefer well-maintained, widely-trusted libraries with few (ideally zero) transitive deps. |
| NFR-DEP-8 | 🟠 | Automated dependency vulnerability scanning runs in CI once implementation begins. |
| NFR-DEP-9 | 🟠 | Document and justify each added dependency (what it's for, why not stdlib/in-house, the version chosen and its release date). |

## Security — `NFR-SEC`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-SEC-1 | 🔴 | Integration and BYO-storage credentials live only on-device (and, if synced, only as client-encrypted data in the user's BYO storage) — never on the DevTriage server. |
| NFR-SEC-2 | 🔴 | All network traffic (provider APIs, CORS proxy, BYO storage) uses TLS. |
| NFR-SEC-3 | 🔴 | On native mobile, sensitive secrets use the platform secure store (iOS Keychain, Android Keystore). |
| NFR-SEC-4 | 🔴 | The web CORS proxy handles tokens only in transit, never persisting or logging them (see [OQ-8a](09-open-questions.md)). |
| NFR-SEC-5 | 🔴 | Content written to BYO storage is encrypted client-side; encryption keys never reach the DevTriage server (see [OQ-24](09-open-questions.md)). |
| NFR-SEC-6 | 🟠 | Tokens can be revoked, and DevTriage requests least-privilege scopes. |
| NFR-SEC-7 | 🟠 | The CORS proxy is protected against abuse (e.g. as an open relay) by requiring an authenticated account ([OQ-22](09-open-questions.md)). |
| NFR-SEC-8 | 🔴 | Authentication is **passwordless** (passkey / Google / GitHub); the server stores **no passwords** — only a public key or provider subject id. |
| NFR-SEC-9 | 🔴 | Encryption uses **platform crypto primitives only** (Web Crypto / CryptoKit / JCA) — never hand-rolled, and no third-party crypto dependency. Content is AES‑256‑GCM under a per-user DEK (OQ-24). |
| NFR-SEC-10 | 🔴 | Encryption keys are **decoupled from login** and recoverable only via a user-held **recovery code**; the server can never recover a user's data (zero-knowledge). This trade-off is clearly communicated to the user (FR-SET-0a). |

## Privacy & data ownership — `NFR-PRIV`

> DevTriage is a **hosted service** (project owner's Hetzner VM), but is designed so the
> **server never holds customer data**. Data lives on the user's devices and in their
> own (BYO) storage.

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-PRIV-1 | 🔴 | The hosted server stores **no customer content** — no documents/notes/todos, no integration tokens, no fetched items. It stores only: (a) **minimal account/identity** records (id, auth identity, timestamps) for login and user-count metrics, (b) **aggregate** usage metrics, and (c) optional **E2EE/opaque sync-coordination metadata** it cannot read (see [06](06-architecture-and-tech-decisions.md), [OQ-26](09-open-questions.md)). A wiped server loses no readable user content. |
| NFR-PRIV-1a | 🔴 | **Account data minimization** — store the least possible for accounts; no content is ever attached to an account, and metrics are aggregate (no per-user content/behavior profiles). |
| NFR-PRIV-2 | 🔴 | User content lives only on the user's devices and in the user's connected **BYO storage**, encrypted client-side so the storage provider cannot read it. |
| NFR-PRIV-3 | 🔴 | No user content is sent to third parties except (a) the integrations the user explicitly connects and (b) the BYO storage the user chooses. |
| NFR-PRIV-4 | 🟠 | The user can export and delete all their data; deletion is effective because nothing is retained server-side. |
| NFR-PRIV-5 | 🟠 | Any optional AI/LLM-assisted features clearly disclose data flow and are opt-in (ties to FR-EXTRACT-7). |

## Performance — `NFR-PERF`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-PERF-1 | 🔴 | Quick-capture is near-instant: an item is saved locally/optimistically within ~3 seconds of intent on any client. |
| NFR-PERF-2 | 🟠 | The unified inbox renders common workloads (hundreds of active items) without noticeable lag. |
| NFR-PERF-3 | 🟠 | Integration sync is incremental and respects rate limits (see INT-COM-7). |

## Reliability & data integrity — `NFR-REL`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-REL-1 | 🔴 | User-captured data is durably stored and not lost on sync conflicts. |
| NFR-REL-2 | 🔴 | A failed integration sync never deletes or corrupts previously collected items. |
| NFR-REL-3 | 🔴 | Sync conflicts (e.g. offline edits) resolve deterministically with no silent data loss — via the pragmatic-hybrid strategy (OQ-23; see [06](06-architecture-and-tech-decisions.md)): per-field LWW, OR-set tags, tombstoned deletes, and conflict copies for divergent long text. |

## Offline & connectivity — `NFR-OFF`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-OFF-1 | 🟠 | Native mobile clients allow capture while offline and sync later (supports FR-SYNC-3). |
| NFR-OFF-2 | 🟢 | Recently-synced data is viewable offline. |

## Portability & deployment — `NFR-PORT`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-PORT-1 | 🔴 | The hosted server is a single, stateless, self-contained unit (single Go binary / container) deployable on one Hetzner VM. |
| NFR-PORT-2 | 🔴 | Because the server is stateless, it can be redeployed or replaced without data migration or backup. |
| NFR-PORT-3 | 🟠 | The BYO-storage adapter layer is pluggable so additional storage backends can be added without client rewrites (see [OQ-21](09-open-questions.md)). |

## Accessibility & UX — `NFR-A11Y`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-A11Y-1 | 🟠 | Clients follow platform accessibility guidelines (contrast, dynamic type, screen-reader labels). |
| NFR-A11Y-2 | 🟢 | Keyboard-driven triage on web. |

## Observability — `NFR-OBS`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-OBS-1 | 🟠 | Backend logs sync activity and errors (without secrets) for troubleshooting. |
| NFR-OBS-2 | 🟢 | Basic health/metrics endpoint for the service operator (no user content in metrics). |

## Maintainability — `NFR-MNT`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-MNT-1 | 🟠 | Connector model is modular so GitHub/Jira/future sources are isolated (supports FR-INT-12). |
| NFR-MNT-2 | 🟠 | Shared API contract is documented and versioned for the three clients. |
