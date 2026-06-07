# 01 — Personas & Use Cases

## Primary persona — "The Individual Developer"

**Who:** A software developer or technical IC who works across GitHub and Jira and
also keeps personal notes/todos.

**Pains:**

- Work is scattered across GitHub notifications, Jira boards, and personal notes.
- Action items in meeting notes get lost.
- No single answer to "what should I work on next?"

**Needs:**

- One inbox of everything assigned to them.
- A fast capture tool for personal todos and notes.
- The same view on desktop and phone.

**Definition of success:** Starts the day in DevTriage, triages everything in a few
minutes, and trusts that nothing assigned to them is missing.

## Secondary persona — "The Self-Hoster"

A privacy-conscious developer who wants to run DevTriage on their own
server/hardware, keep credentials local, and avoid sending data to third parties.

> Team/manager personas are explicitly **out of scope** for the initial product
> (see [Vision & Scope](00-vision-and-scope.md)).

## Core use cases

### UC-1 — Capture a personal todo
The user quickly adds a todo (title, optional due date/priority/tags) from any client
and it appears everywhere. → `FR-CAP`

### UC-2 — Write a note / idea
The user writes free-form markdown notes or quick ideas, with no required structure.
→ `FR-NOTE`

### UC-3 — Auto-collect todos from a note
The user writes a note containing action items (e.g. checkbox lines or
imperative sentences). DevTriage detects them and offers/creates linked todos. →
`FR-EXTRACT`

### UC-4 — See everything assigned to me
After connecting GitHub and Jira, the user opens one inbox showing all issues, PRs,
review requests, and tickets assigned to them. → `FR-INT`, `FR-TRIAGE`

### UC-5 — Triage the inbox
The user prioritizes, snoozes, groups, searches, and completes items — whether they
originated locally or from an external system. → `FR-TRIAGE`

### UC-6 — Work across devices
The user captures on mobile during a meeting and triages on the web later; state is
consistent. → `FR-SYNC`

### UC-7 — Connect / manage integrations
The user adds, tests, and revokes GitHub and Jira connections and controls what gets
collected. → `FR-INT`, `FR-SET`

## Primary daily flow (narrative)

1. Morning: open DevTriage → unified inbox shows new GitHub PRs to review, a Jira
   ticket reassigned overnight, and 2 todos auto-extracted from yesterday's notes.
2. Triage: snooze one PR to the afternoon, raise priority on the ticket, mark a
   stale todo done.
3. During the day: capture quick todos/ideas from the phone.
4. End of day: a meeting note's action items appear as suggested todos to confirm.
