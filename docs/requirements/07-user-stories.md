# 07 — User Stories & Acceptance Criteria

Acceptance criteria are written in Given/When/Then form for the highest-priority
requirements. Each story links to the requirement(s) it validates.

---

## Capture

### US-1 — Quickly capture a todo · `FR-CAP-1`, `FR-CAP-4`, `NFR-PERF-1`
*As a developer, I want to capture a todo in a couple of taps/keystrokes so a thought
isn't lost.*

- **Given** I'm on any client, **when** I open quick capture and type a title and
  confirm, **then** the todo is saved and visible in my inbox.
- **Given** I have no connection, **when** I capture on mobile, **then** it is stored
  locally and syncs later (ties to `FR-SYNC-3`).
- Capture-to-saved feels instant (optimistic save within ~3s).

### US-2 — Add detail to a todo · `FR-CAP-2`, `FR-CAP-3`
*As a developer, I want to add due date, priority, tags, and a project so I can
organize work.*

- **Given** an existing todo, **when** I edit its fields, **then** changes persist and
  sync to other clients.
- **When** I complete it, **then** it leaves the active inbox and can be reopened.

---

## Notes & extraction

### US-3 — Write a note · `FR-NOTE-1`, `FR-NOTE-3`
*As a developer, I want free-form Markdown notes so I can jot meeting/design notes.*

- **Given** the editor, **when** I write Markdown and save, **then** the note persists
  and is searchable (`FR-NOTE-5`).

### US-4 — Auto-collect todos from a note · `FR-EXTRACT-1..4`
*As a developer, I want action items in my notes to become todos automatically so I
don't re-enter them.*

- **Given** a note with `- [ ]` lines, **when** I save it, **then** each checkbox is
  detected as a candidate todo.
- **Given** my mode is *suggest-and-confirm*, **then** candidates are shown for me to
  accept/reject; **given** *auto-create*, **then** they become todos immediately.
- **Given** an extracted todo, **when** I open it, **then** I can navigate back to its
  source note and line (`FR-EXTRACT-3`).
- **Given** I check the box in the source note, **then** the linked todo is completed
  (`FR-EXTRACT-6`, SHOULD).

---

## Integrations

### US-5 — Connect GitHub · `FR-INT-1`, `INT-GH-*`, `INT-COM-2`
*As a developer, I want to connect GitHub so my assigned work shows up.*

- **Given** the integrations screen, **when** I enter a credential and test it,
  **then** DevTriage confirms it works and shows my GitHub identity.
- **After** the first sync, **then** issues assigned to me, PRs requesting my review,
  and my open PRs appear in the inbox with deep links (`INT-GH-1..3`, `FR-INT-8`).

### US-6 — Connect Jira · `FR-INT-2`, `INT-JIRA-*`
*As a developer, I want to connect Jira so my tickets show up.*

- **After** connecting, **then** issues assigned to me in non-done statuses appear in
  the inbox (`INT-JIRA-1`).
- **Given** I provide custom JQL, **then** collection uses it (`INT-JIRA-3`, SHOULD).

### US-7 — Stay in sync with the source · `FR-INT-7`, `FR-INT-9`, `INT-COM-5/8`
*As a developer, I want external items to reflect reality so my inbox stays trustworthy.*

- **When** a sync runs (scheduled or manual), **then** new items appear and no
  duplicates are created.
- **When** an item is closed/resolved at the source, **then** it reflects as
  closed/done in DevTriage on the next sync (SHOULD).
- **When** a sync fails (bad token/rate limit), **then** I see an error and my
  existing items are preserved (`INT-COM-6`, `NFR-REL-2`).

---

## Triage

### US-8 — Triage everything in one place · `FR-TRIAGE-1..5`
*As a developer, I want one inbox of personal todos and assigned work so I know what
to do next.*

- **Given** todos + GitHub + Jira items, **then** all appear in one list.
- **When** I filter/sort/group/search, **then** results span all sources.
- **When** I complete/dismiss an item, **then** it leaves the active inbox.

### US-9 — Add my own context to assigned work · `FR-TRIAGE-7`
*As a developer, I want to tag/prioritize/snooze an assigned item locally without
changing the source.*

- **Given** an external item, **when** I add a local tag/priority/snooze, **then** it
  applies in DevTriage only and is not written back to GitHub/Jira.

---

## Cross-device

### US-10 — Work across devices · `FR-SYNC-1/2`
*As a developer, I want consistent state on web and phone.*

- **Given** I capture on mobile, **when** I open web, **then** the item is there after
  sync, and vice-versa.

---

## Settings & security

### US-11 — Manage credentials safely · `FR-SET-2/3`, `NFR-SEC-1/4`
*As a security-conscious user, I want my tokens stored safely and revocable.*

- **When** I add a credential, **then** it is stored encrypted and never appears in
  logs.
- **When** I revoke a connection, **then** syncing stops and the credential is removed.
