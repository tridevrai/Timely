# Bolt Plan: Timely

A **Bolt** is one build pass over a piece of the work: design, code and tests for
one unit, ending in a working app that the owner approves before the next Bolt
starts. Timely has five Bolts, one per unit [Q3], built one after another [Q4],
in "most useful first" order after the walking skeleton [Q1]. The order is
argued in prose, not scored [Q2]; see `risk-and-sequencing-rationale.md`.

Units, stories and dependencies come from `units-generation/`. Practices from
`team.md`: short-lived branches squash-merged into `main` with CI blocking the
merge; a walking skeleton first; test-after with at least 80% line coverage.

| Order | Bolt | Unit | Stories | Walking skeleton |
|---|---|---|---|---|
| 1 | B1 Core loop | U1 `u1-core-loop-slice` | 8 | **Yes** |
| 2 | B2 Sort and lists | U2 `u2-sort-and-lists` | 9 | |
| 3 | B3 Day and focus | U3 `u3-day-and-focus` | 7 | |
| 4 | B4 Calendar sync | U4 `u4-calendar-sync` | 4 | |
| 5 | B5 Reminders and insights | U5 `u5-reminders-and-insights` | 4 | |

The order respects the unit dependency map: U1 has no dependencies; U2, U3 and
U4 need only U1; U5 needs U1 and U3. It is one of the valid topological orders,
so no deviation needs justifying.

**Use-it gate between Bolts** (owner's main worry, "productivity theater" [Q7]):
each Bolt's Definition of Done includes the owner actually planning at least one
real day with the app as it stands before the next Bolt starts. If the core loop
alone turns out to be enough, later Bolts can be dropped or reordered at that point.

---

## B1 Core loop: the walking skeleton (U1)

- **Walking skeleton**: the smallest version of Timely that runs the whole core
  loop end to end. It proves the architecture: the single app serving pages and
  API (ADR-009), session sign-in, the transactional outbox (C2), the Tasks →
  Planning → Focus → Notifications path, and web push to a real iPhone.
- **Stories**: US1.1, US2.1, US2.2, US3.1, US4.1, US5.1, US6.1, US7.1.
- **Prerequisites**: the technology stack chosen in Construction design stages;
  a reachable server with a domain name and HTTPS before the end-to-end check [Q6];
  a git repository with `main` and CI (team.md Way of Working).
- **Definition of Done**:
  - All eight stories' acceptance criteria pass as automated tests; ≥ 80% line coverage; lint clean; CI green on the merge into `main`.
  - Deployed to the HTTPS server; the app installs to the iPhone home screen.
  - The real end-to-end check passes (below).
  - The owner plans one real day with it.
- **Confidence hypothesis**: the core loop works on a real iPhone and Mac, from
  brain dump to a notification arriving in the background, and the owner finds
  it useful enough to keep using.
- **Expected demo**: on the iPhone, sign in → paste 5 lines into Capture → do one
  as a 2-minute task → sort one into Do Now → give it a 25-minute block today on
  the Mac (drag) → start a Pomodoro → lock the phone → the end-of-interval
  notification arrives with sound.
- **Real integrated check**: an automated end-to-end test that runs the same flow
  against the running app in a real browser (with notification delivery checked
  at the push-send boundary), plus the manual iPhone notification check above.

## B2 Sort and lists (U2)

- **Stories**: US1.2, US3.2, US4.2–US4.7, US11.1.
- **Definition of Done**: all acceptance criteria automated; coverage, lint and CI as B1; deployed; the owner sorts a real brain dump using all four categories and handles one follow-up.
- **Confidence hypothesis**: sorting a real inbox one card at a time takes under 2 minutes for 10 items, and deferred and delegated items come back on the right day in the owner's time zone.
- **Expected demo**: sort 10 items across Do Now, Delegate, Defer and Drop; undo a drop; on the follow-up day, "Push date" and "Take it back"; let a session expire mid-capture and get the text back.

## B3 Day and focus (U3)

- **Stories**: US5.2, US5.3, US6.2, US6.3, US6.4, US7.2, US11.2.
- **Definition of Done**: as B2; plus the timer stays within 2 seconds after 10 minutes of phone sleep in a manual test.
- **Confidence hypothesis**: the timer and the end-of-block decisions hold up over a real working day on two devices, and edits never silently overwrite each other.
- **Expected demo**: overlapping blocks flagged; pause and resume a Pomodoro; reload mid-interval; finish a block with "Needs more time"; edit the same task on Mac and iPhone and get the "changed elsewhere" choice.

## B4 Calendar sync (U4)

- **Stories**: US8.1, US8.2, US8.3, US8.4.
- **Prerequisites**: the owner's Google Cloud project with an OAuth consent screen in testing status, created before B4 starts [Q5].
- **Definition of Done**: as B2, with a fake Google adapter in automated tests; plus the owner's manual check against their real Google Calendar: connect, see today's events, create/move/delete a block and see it mirrored, move it in Google and see it come back, disconnect Wi-Fi and reconnect with no duplicate events.
- **Confidence hypothesis**: two-way sync never creates duplicates or loses a change across outages and conflicts, and the weekly reconnect of a testing-status Google app is tolerable.
- **Expected demo**: the manual check above.

## B5 Reminders and insights (U5)

- **Stories**: US9.1, US9.2, US10.1, US10.2.
- **Definition of Done**: as B2; one full week of real use before the summary is judged.
- **Confidence hypothesis**: the weekly summary shows whether the app is working (planned ≥ 5 days a week, most blocks done), and the reminders help batch messages without becoming noise.
- **Expected demo**: message reminders at the set times; the verse once per day; the weekly summary for the past week with the open-list limit.
