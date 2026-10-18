# Architecture Decisions: Domain Design

All ADRs: Status Accepted pending the Domain Design approval gate, dated
2026-10-19. Sources are the answers in `domain-design-questions.md`.

## ADR-001: One Tasks component owns the whole task lifecycle

- **Context**: A task moves between unsorted, Do Now, Delegate, Defer, Drop and
  completed, often several times (US4.5). The owner wants one place where a task
  lives. [Q1]
- **Decision**: One Tasks component owns the Task entity; the category is a
  state of the task. Moving categories is a single versioned update.
- **Consequences**: (+) one transaction per change, no hand-off bugs, simple
  lists and counts. (+) Delegate/Defer rules sit together. (−) Tasks is the
  largest component; its internal state machine must be designed carefully in
  Functional Design.
- **Alternatives Rejected**: Separate Inbox, Delegation and Deferral
  components: each move would be a cross-component hand-off that can half-fail,
  for no independent change rate.
- **Security/compliance**: every task query is scoped by ownerId.

## ADR-002: All countdowns live in Focus

- **Context**: Pomodoro, the 2-minute countdown and the 5-minute start all need
  timers that survive sleep, background and reload (FR8.3). [Q2]
- **Decision**: Focus owns all three. Remaining time is derived from a recorded
  start instant and paused duration. Focus calls Tasks to complete a task.
- **Consequences**: (+) one correct timer implementation, one set of tests.
  (−) the 2-minute flow spans Focus and Tasks.
- **Alternatives Rejected**: 2-minute countdown in Tasks: duplicates the timer
  logic and its edge cases.
- **Security/compliance**: none beyond owner scoping.

## ADR-003: Time blocks are separate from tasks

- **Context**: A task may need more than one block in a day ("needs more time",
  FR9.2), and blocks are what sync to Google. [Q3]
- **Decision**: Planning owns TimeBlock; each block references one task.
- **Consequences**: (+) several blocks per task; tasks without blocks are
  natural; sync works on blocks only. (−) Planning must react when a task
  leaves Do Now or is completed (via Tasks events, ADR-010).
- **Alternatives Rejected**: Start/end on the task: one slot per task, and
  calendar sync would couple to the task model.
- **Security/compliance**: blocks scoped by ownerId; a block may only reference
  the same owner's task.

## ADR-004: CalendarSync is an anti-corruption layer; newest change wins

- **Context**: Two-way sync with Google Calendar (FR7) brings an external model,
  OAuth tokens, rate limits and outages. Conflicts are resolved by the newest
  change (stories Q6). [Q4]
- **Decision**: A separate CalendarSync component owns the Google connection,
  the block-to-event links and conflict notices. It listens to Planning's block
  events and applies Google-side changes through Planning's interface. When both
  sides changed since the last sync, the later modification time wins and the
  owner is told.
- **Consequences**: (+) Planning and the rest never see Google concepts; Google
  outages degrade only sync. (+) Tokens are confined to one component. (−) Sync
  is eventually consistent (within the sync interval). (−) Comparing modification
  times across systems depends on clock accuracy; clock skew is a known risk
  noted for NFR Design.
- **Alternatives Rejected**: Planning calling Google directly: leaks Google's
  model and failure modes into planning. Timely-always-wins or Google-always-wins:
  rejected by the owner in User Stories Q6.
- **Security/compliance**: OAuth tokens stored encrypted, never logged (NFR4);
  minimum calendar scope requested; revoking access must not lose blocks.

## ADR-005: One Notifications component delivers every alert

- **Context**: Block ends, Pomodoro phases and message windows all notify the
  owner, on Mac and iPhone, in the background (FR9, FR10). [Q5]
- **Decision**: Notifications owns device subscriptions and a schedule of
  alerts keyed by source, which Planning and Focus schedule and cancel. It builds
  message-window reminders from Account preferences.
- **Consequences**: (+) one delivery path, one retry policy, no duplicate
  alerts. (−) Callers depend on it being available to schedule; scheduling
  failures must be visible.
- **Alternatives Rejected**: Each component sending its own notifications:
  duplicates subscription handling and makes duplicates likely.
- **Security/compliance**: alert payloads contain only a short title; no task
  text in delivery logs (NFR8).

## ADR-006: Weekly summary computed on request

- **Context**: The summary is read rarely and the data is small (one owner). [Q6]
- **Decision**: Insights reads from Tasks and Planning each time and stores
  nothing.
- **Consequences**: (+) always consistent with the source data; nothing to
  migrate. (−) cost grows with history; acceptable at single-user scale
  (NFR2 sets the item bound).
- **Alternatives Rejected**: Running totals: risk of drift, extra writes on
  every change.
- **Security/compliance**: none beyond owner scoping.

## ADR-007: Preferences live in Account

- **Context**: Time zone, message windows and open-list limit are owner-level
  settings read by several components. [Q7]
- **Decision**: Account owns them on the Owner entity and emits
  PreferencesChanged.
- **Consequences**: (+) one settings screen backed by one component. (−)
  Several components read Account, making it a hub; it is the most stable part.
- **Alternatives Rejected**: A separate Settings component (no distinct
  behaviour); settings spread across users (Notifications, Insights): the
  settings screen would talk to many places.
- **Security/compliance**: time zone validated against the IANA list.

## ADR-008: Verses stored on the server, no editing screen

- **Context**: The owner wants the verse list changeable without a release, but
  version 1 avoids extra settings. [Q8, follow-up 1]
- **Decision**: A small Verses component holds a seeded list and the per-day
  dismissed state. There is no editing screen; changes are made in the data.
- **Consequences**: (+) list changes need no release. (−) one more small
  component and table. This changes the requirements assumption that verses are
  a fixed list in the app; the feature (FR11) is unchanged.
- **Alternatives Rejected**: Fixed list in the Web App (owner's choice, Q8);
  an editing screen (owner's choice, follow-up 1; out of scope per Q14).
- **Security/compliance**: verse text rendered as plain text.

## ADR-009: The server decides every rule

- **Context**: The same data is used from a Mac and an iPhone; edits must never
  silently overwrite each other (FR13.3). [Q9]
- **Decision**: Server components are authoritative. Every entity that can be
  edited from two devices (Task, TimeBlock) carries a version; a stale update is
  rejected and the Web App asks the owner which version to keep. The Web App
  validates while typing only for feedback. Without the server, the app shows it
  is offline; typed drafts are kept in the browser.
- **Consequences**: (+) one source of truth, simple conflict rule, rules tested
  once on the server. (−) No offline editing in version 1.
- **Alternatives Rejected**: Offline-first client with later sync: much more
  complex conflict handling for a single-user app.
- **Security/compliance**: every input validated on the server (project.md
  Mandated rule).

## ADR-010: Events break dependency cycles

- **Context**: Planning needs to know when a task changes category, and
  CalendarSync needs to know when blocks change, but Tasks and Planning should
  not call their dependents.
- **Decision**: Tasks emits TaskCategoryChanged and TaskCompleted; Planning
  emits BlockCreated, BlockChanged and BlockRemoved; Account emits
  PreferencesChanged. Dependents subscribe. The dependency graph stays acyclic.
- **Consequences**: (+) clear direction of dependency. (−) event delivery must be
  reliable inside the system (no lost events); the mechanism is decided in
  Units Generation and NFR Design.
- **Alternatives Rejected**: Direct calls both ways: creates Tasks↔Planning and
  Planning↔CalendarSync cycles.
- **Security/compliance**: events stay internal and carry IDs, not task text.
