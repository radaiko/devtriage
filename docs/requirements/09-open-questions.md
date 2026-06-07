# 09 — Open Questions

Decisions still to be made. Each should be resolved before (or early in) the relevant
phase. Owner = project owner unless stated.

## Architecture & stack

| # | Question | Notes / leaning |
| --- | --- | --- |
| OQ-1 | **Backend language** — confirm **Go**? | Leaning Go: single binary, std-lib heavy, no npm, great concurrency (see [06](06-architecture-and-tech-decisions.md#backend--proposed-go)). Alt: Rust. |
| OQ-2 | **Web client approach** under the npm-avoidance constraint. | Leaning server-rendered / minimal-JS to honor [NFR-DEP](04-non-functional-requirements.md). Trade-off: interactivity. |
| OQ-3 | **Storage** — SQLite by default, Postgres optional? | Leaning SQLite for single-user self-host. |
| OQ-4 | **API style** — REST (leaning) vs gRPC vs GraphQL. | REST is simplest for Swift/Kotlin/web. |
| OQ-5 | **Real-time updates** now or later? | Proposed: poll/refresh first, push later (FR-SYNC-4). |

## Integrations

| # | Question | Notes |
| --- | --- | --- |
| OQ-6 | **GitHub auth** — fine-grained PAT vs OAuth vs GitHub App? | Affects scopes, setup friction, multi-account. |
| OQ-7 | **Jira auth** — API token (Basic) vs OAuth? Cloud only first? | Cloud first; Server/DC later. |
| OQ-8 | **Polling vs webhooks** for freshness. | Webhooks reduce latency/rate-limit pressure but complicate self-hosting (inbound reachability). |
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
| OQ-16 | **Single-user only** initially, or design auth for multi-user from day one? | Scope says single-user; confirm. |
| OQ-17 | **Hosting model** — self-host only, or also an optional hosted offering? | Affects security/privacy requirements. |
| OQ-18 | **Minimum OS versions** for iOS/Android targets. | Influences SwiftUI/Compose API availability. |

## Project

| # | Question | Notes |
| --- | --- | --- |
| OQ-19 | **License** for the repository. | README currently says TBD. |
| OQ-20 | Repository structure once coding begins — **monorepo** (backend + web + iOS + android) vs separate repos. | Monorepo likely simplest for a solo/small effort. |

---

> When an item is decided, record the decision in the relevant requirements/architecture
> doc and mark it resolved here (don't delete the history of why).
