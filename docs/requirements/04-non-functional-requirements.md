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
| NFR-DEP-3 | 🔴 | Every third-party dependency must be deliberately chosen, pinned to an exact version, and integrity-checked (lockfile / checksums / vendoring). |
| NFR-DEP-4 | 🟠 | Dependencies should be auditable; prefer well-maintained, widely-trusted libraries with few transitive deps. |
| NFR-DEP-5 | 🟠 | Automated dependency vulnerability scanning runs in CI once implementation begins. |
| NFR-DEP-6 | 🟠 | Document and justify each added dependency (what it's for, why not stdlib). |

## Security — `NFR-SEC`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-SEC-1 | 🔴 | Integration credentials (GitHub/Jira tokens) are stored encrypted at rest. |
| NFR-SEC-2 | 🔴 | Credentials and synced content are transmitted only over TLS. |
| NFR-SEC-3 | 🔴 | On native mobile, sensitive secrets use the platform secure store (iOS Keychain, Android Keystore). |
| NFR-SEC-4 | 🔴 | Credentials are never written to logs or committed to the repo. |
| NFR-SEC-5 | 🟠 | Tokens can be revoked, and DevTriage requests least-privilege scopes. |
| NFR-SEC-6 | 🟠 | Authentication to DevTriage itself follows current best practice (hashed credentials, session/token expiry). |

## Privacy & data ownership — `NFR-PRIV`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-PRIV-1 | 🔴 | DevTriage is self-hostable; the user can run the backend under their own control. |
| NFR-PRIV-2 | 🔴 | No user content is sent to third parties except the integrations the user explicitly connects. |
| NFR-PRIV-3 | 🟠 | The user can export and delete all their data. |
| NFR-PRIV-4 | 🟠 | Any optional AI/LLM-assisted features clearly disclose data flow and are opt-in (ties to FR-EXTRACT-7). |

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
| NFR-REL-3 | 🟠 | Sync conflicts (e.g. offline edits) resolve deterministically with no silent data loss. |

## Offline & connectivity — `NFR-OFF`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-OFF-1 | 🟠 | Native mobile clients allow capture while offline and sync later (supports FR-SYNC-3). |
| NFR-OFF-2 | 🟢 | Recently-synced data is viewable offline. |

## Portability & deployment — `NFR-PORT`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-PORT-1 | 🔴 | The backend is deployable as a self-contained unit on commodity hardware (single binary / container preferred). |
| NFR-PORT-2 | 🟠 | Storage defaults to a simple, low-operations option suitable for single-user self-hosting. |

## Accessibility & UX — `NFR-A11Y`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-A11Y-1 | 🟠 | Clients follow platform accessibility guidelines (contrast, dynamic type, screen-reader labels). |
| NFR-A11Y-2 | 🟢 | Keyboard-driven triage on web. |

## Observability — `NFR-OBS`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-OBS-1 | 🟠 | Backend logs sync activity and errors (without secrets) for troubleshooting. |
| NFR-OBS-2 | 🟢 | Basic health/metrics endpoint for self-hosters. |

## Maintainability — `NFR-MNT`

| ID | Priority | Requirement |
| --- | --- | --- |
| NFR-MNT-1 | 🟠 | Connector model is modular so GitHub/Jira/future sources are isolated (supports FR-INT-12). |
| NFR-MNT-2 | 🟠 | Shared API contract is documented and versioned for the three clients. |
