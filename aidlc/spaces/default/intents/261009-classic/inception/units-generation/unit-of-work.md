# Units of Work: Timely

Built from `domain-design/components.md` (9 components, ADR-001 to ADR-010),
`requirements.md` and `stories.md` (32 stories), with the answers in
`units-generation-questions.md`: one deployable app [Q1], feature-slice units
[Q2], five units [Q3]. The owner approved this plan before generation.

## Deployment model

Timely ships as **one deployable app**: a single server process that serves the
web pages and the API, on one self-hosted server [Q1]. The nine components stay
separate modules inside it (a modular monolith). Modules call each other only
through their defined interfaces, never another module's data, and the internal
events from ADR-010 are delivered in-process. Every unit below is a slice of
work through that same app; no unit is deployed on its own. This keeps open
the option of extracting a module later (for example CalendarSync).

## Units

| Unit ID | Directory | Name | Kind | Deployment | Complexity |
|---|---|---|---|---|---|
| U1 | u1-core-loop-slice | Core loop slice | service | shared (the one app) | L |
| U2 | u2-sort-and-lists | Sort and lists | service | shared | M |
| U3 | u3-day-and-focus | Day and focus | service | shared | M |
| U4 | u4-calendar-sync | Calendar sync | service | shared | L |
| U5 | u5-reminders-and-insights | Reminders and insights | service | shared | S |

`service` means each unit is screens plus server code that runs, so each gets the
full construction design treatment.

---

### U1 Core loop slice (`u1-core-loop-slice`) — the walking skeleton

- **Description**: The smallest version of Timely that runs the whole core loop
  for real: sign in → brain dump → 2-minute rule → sort into Do Now → time-block
  it today → Pomodoro → end notification (team.md Walking Skeleton).
- **Boundaries**: Only the paths these eight stories need. Other sort categories,
  calendar sync, overlaps and summaries are later units.
- **Responsibilities** (components touched):
  - Account: the single owner, sign-in, sessions, time zone preference.
  - Tasks: capture with idempotency, unsorted list, 2-minute completion, Do Now.
  - Planning: create a block for a Do Now task today; block-end alert.
  - Focus: Pomodoro (25/5/15) and the 2-minute countdown, computed from start time.
  - Notifications: device subscription and delivery of interval and block-end alerts.
  - WebApp: Sign in, Capture, Sort (Do Now and 2-minute only), Today timeline,
    schedule sheet, Focus view, tab bar, Mac two-panel layout with keyboard scheduling.
  - App shell: the single server, database setup and migrations, configuration
    from environment variables (no secrets in code), HTTPS, CI with tests,
    coverage and lint.
- **Stories**: US1.1, US2.1, US2.2, US3.1, US4.1, US5.1, US6.1, US7.1.
- **Complexity**: L. It carries the app shell and the first piece of every
  core component.
- **Implementation notes**: The verification for this unit must show the real
  end-to-end result: a signed-in owner captures, sorts, blocks, focuses and
  receives a notification on a real browser. Data-model choices made here
  (task versioning, owner scoping) are kept by every later unit.

### U2 Sort and lists (`u2-sort-and-lists`)

- **Description**: The rest of the sort and everything about lists.
- **Boundaries**: Tasks and its screens; Planning only for removing blocks when a
  task leaves Do Now (via the TaskCategoryChanged event).
- **Responsibilities**: Delegate (name, follow-up date), Defer (future date,
  midnight return), Drop with Undo and Restore, changing category, next-step
  prompt, follow-up reminders and their actions, cancelling the 2-minute
  countdown, list ordering and empty states, browser drafts kept through
  session expiry.
- **Stories**: US1.2, US3.2, US4.2, US4.3, US4.4, US4.5, US4.6, US4.7, US11.1.
- **Complexity**: M.
- **Implementation notes**: Midnight return of deferred tasks uses the owner's
  time zone and must not run twice across a daylight-saving change.

### U3 Day and focus (`u3-day-and-focus`)

- **Description**: Makes the day plan and the timers robust.
- **Boundaries**: Planning, Focus and the matching WebApp states; Tasks only
  through its interface.
- **Responsibilities**: overlap flagging, carry-over of unscheduled Do Now
  tasks, timer accuracy through sleep and reload, pause/resume/stop, the
  5-minute start, end-of-block decisions (Done, Needs more time, Reschedule),
  and the Mac/iPhone edit conflict prompt using entity versions (ADR-009).
- **Stories**: US5.2, US5.3, US6.2, US6.3, US6.4, US7.2, US11.2.
- **Complexity**: M.
- **Implementation notes**: US11.2's version checks touch both Task and
  TimeBlock; the rule is shared, so it is built once here and used by later units.

### U4 Calendar sync (`u4-calendar-sync`)

- **Description**: Two-way Google Calendar sync behind the CalendarSync
  anti-corruption layer (ADR-004).
- **Boundaries**: CalendarSync plus the connect flow, events on the timeline,
  sync state and conflict notices in WebApp. Planning is used only through its
  interface and block events.
- **Responsibilities**: OAuth connection and encrypted tokens, reading today's
  events, writing/updating/removing one event per block, picking up Google-side
  changes, newest-change-wins conflicts with a notice, offline and
  revoked-access handling with retries that never duplicate events.
- **Stories**: US8.1, US8.2, US8.3, US8.4.
- **Complexity**: L. It depends on an external API with outages, rate limits and
  token expiry.
- **Implementation notes**: Tests use a fake Google adapter; one manual check
  against a real Google calendar before the unit is approved.

### U5 Reminders and insights (`u5-reminders-and-insights`)

- **Description**: The supporting habits around the core loop.
- **Boundaries**: Notifications (message windows), Verses, Insights, and the
  Settings and Summary screens.
- **Responsibilities**: message-window reminders from preferences, the daily
  verse (seeded server list, dismissed per day), the weekly summary computed on
  request, the open-list limit setting.
- **Stories**: US9.1, US9.2, US10.1, US10.2.
- **Complexity**: S.
- **Implementation notes**: The summary counts "done" blocks, so it needs U3's
  end-of-block decisions.
