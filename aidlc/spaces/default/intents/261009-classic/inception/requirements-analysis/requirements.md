# Requirements: Timely

## Intent Analysis

Timely is a personal planning app that takes its owner from overwhelm to focused
execution. It replaces "the massive list" with one daily loop: get everything out
of your head, sort it, give the chosen work a time slot today, then work that
slot in focused Pomodoro intervals.

- **Goal**: reduce background anxiety from open tasks and turn the day into a
  small set of time-blocked, finishable commitments. [doc §1.1, §1.3]
- **Primary user**: the owner, a single person working across Mac and iPhone.
  Other people appear only as names on delegated tasks in this version. [Q2, FU1]
- **Guiding principle**: avoid "productivity theater". The first version is the
  core loop and nothing else. [doc §3.2, Q14, team.md Walking Skeleton]
- **Type / complexity**: new product, multi-component (web client, server,
  Google Calendar integration), Standard depth.

### Core loop (the walking skeleton)

Brain dump → 2-minute rule → sort into Do Now / Delegate / Defer / Drop →
time-block Do Now items on today's plan → Pomodoro focus session in a block.
This slice is built and approved before any other feature. [team.md Walking Skeleton]

## Functional Requirements

### FR1: Account and access

- **FR1.1** The system shall require the owner to sign in before any task,
  plan, or calendar data is shown or changed. [Q3: server-hosted, reachable from any device]
- **FR1.2** The system shall support exactly one owner account in this version;
  the data model shall keep an owner on every record so more accounts can be added later. [Q2, FU1]
- **FR1.3** When a session expires mid-action, the system shall keep the unsaved
  input and restore it after the owner signs in again.

Acceptance criteria:
- Given I am signed out, When I open any app page, Then I see the sign-in screen and no task data.
- Given my session expired while typing a brain dump, When I sign back in, Then the text I typed is still there.

### FR2: Brain dump capture

- **FR2.1** The system shall provide a single capture screen where the owner can
  enter many items quickly, one per line, without choosing a category first. [doc §1.1]
- **FR2.2** Each non-empty line shall become one unsorted item; blank or
  whitespace-only lines shall be ignored.
- **FR2.3** Captured items shall be saved as soon as they are entered, so closing
  the app never loses them.
- **FR2.4** Submitting the same capture twice (double tap, retry) shall not create duplicate items.

Acceptance criteria:
- Given the capture screen, When I paste 12 lines including 2 blank lines, Then 10 unsorted items are created.
- Given I typed 3 items, When I close the browser tab and reopen the app, Then all 3 items are in my unsorted list.
- Given an item containing emoji or `<script>` text, When it is shown anywhere, Then it is displayed as plain text and never executed.

### FR3: 2-minute rule

- **FR3.1** For each unsorted item, the owner shall be able to mark it "2-minute". [Q7]
- **FR3.2** Starting a 2-minute item shall show a 2-minute countdown; when it
  ends, the system shall ask whether the item is done. [Q7 B]
- **FR3.3** If done, the item is marked complete. If not done, it returns to
  the unsorted list to be sorted normally.
- **FR3.4** 2-minute items never require a sort category.

Acceptance criteria:
- Given an unsorted item, When I start it as 2-minute, Then a 2:00 countdown runs.
- Given the countdown reached 0:00, When I answer "Done", Then the item is complete and leaves the unsorted list.
- Given the countdown reached 0:00, When I answer "Not yet", Then the item is back in the unsorted list.

### FR4: Sorting (Do Now / Delegate / Defer / Drop)

- **FR4.1** The owner shall be able to sort every unsorted item into exactly one
  of: Do Now, Delegate, Defer, Drop. [doc §1.1, Q6 A]
- **FR4.2** **Do Now**: the item becomes available to time-block today.
- **FR4.3** **Delegate**: the owner must enter the person's name and a follow-up
  date. Delegated items appear in a Delegated list ordered by follow-up date,
  and on their follow-up date they appear on that day's plan as a follow-up reminder. [Q6 A, FU1 C]
- **FR4.4** **Defer**: the owner must pick a future date. A deferred item is
  hidden until that date, then reappears in the unsorted list. [Q6 A]
- **FR4.5** **Drop**: the item is archived, not deleted. Archived items can be
  viewed and restored to the unsorted list. [Q6 A]
- **FR4.6** The owner shall be able to change an item's category later.

Acceptance criteria:
- Given an unsorted item, When I choose Delegate without a name or date, Then I see which field is missing and the item stays unsorted.
- Given an item deferred to 2026-11-03, When the owner's local date becomes 2026-11-03, Then the item is in the unsorted list.
- Given a dropped item, When I restore it from the archive, Then it is back in the unsorted list with its text unchanged.
- Given a Defer date in the past or today, When I save, Then I am told to choose a future date.

### FR5: Micro-stepping and the 5-minute start

- **FR5.1** When an item is sorted into Do Now and has no next step, the system
  shall prompt the owner for its smallest physical next step; the owner may skip the prompt. [doc §1.2, Q8 A]
- **FR5.2** The next step shall be shown with the item everywhere it appears in the day plan.
- **FR5.3** The system shall offer a 5-minute "just start" timer on any Do Now
  item; when it ends, the owner chooses to keep going (start a Pomodoro) or stop. [doc §1.2, Q8 A]

Acceptance criteria:
- Given I sort "Work on taxes" into Do Now, When the prompt appears, Then I can enter "Log into portal and download June statement" or skip.
- Given the 5-minute timer ends, When I choose "Keep going", Then a Pomodoro session starts for that item.

### FR6: Time-blocking today

- **FR6.1** The system shall show today's plan as a timeline in the owner's
  time zone. Only today can be planned. [Q5 A, doc §3.3]
- **FR6.2** The owner shall be able to place a Do Now item into a time block with
  a start and end time; a block must end after it starts and within today.
- **FR6.3** Existing Google Calendar events for today shall appear on the timeline
  (see FR7) so blocks can be planned around them.
- **FR6.4** Overlapping blocks shall be allowed but visibly flagged.
- **FR6.5** Do Now items not given a block by the end of the day shall stay in
  Do Now for the next day, rather than disappear.

Acceptance criteria:
- Given a Do Now item, When I place it 09:00–10:30 today, Then it appears on today's timeline at that time.
- Given a block that ends before it starts, When I save, Then I see an error and the block is not saved.
- Given an unblocked Do Now item at midnight, When I open the app the next day, Then it is still in Do Now.

### FR7: Google Calendar two-way sync

- **FR7.1** The owner shall connect one Google account; the system shall read
  today's events from the owner's chosen Google calendar. [Q4 C, FU2 B]
- **FR7.2** Each time block created in Timely shall be written to that Google
  calendar as an event, and moving, resizing, or deleting the block shall update or remove that event.
- **FR7.3** Changes made in Google Calendar to an event that Timely created
  shall be reflected back on the Timely block within 5 minutes.
- **FR7.4** When Google Calendar cannot be reached, the app shall keep working
  with Timely's own blocks, show that the calendar is out of date and when it last
  synced, and retry automatically without creating duplicate events.
- **FR7.5** When the Google connection is revoked or expires, the system shall
  tell the owner and offer to reconnect; no blocks shall be lost.

Acceptance criteria:
- Given a connected calendar with a 13:00 meeting, When I open today's plan, Then the meeting shows at 13:00 as read-only.
- Given I create a 09:00–10:00 block, When sync completes, Then one matching event exists in Google Calendar.
- Given Google is unreachable, When I create a block, Then it is saved in Timely, marked "not yet synced", and appears in Google once only after connectivity returns.

### FR8: Pomodoro focus session

- **FR8.1** The owner shall be able to start a Pomodoro session from a time
  block; the session is linked to that block's item. [doc §1.3, Q9]
- **FR8.2** Version 1 cycle: 25 minutes focus, 5 minutes break, and a 15-minute
  long break after every 4 focus intervals. Lengths are fixed in this version. [Q9: B now, C later]
- **FR8.3** The timer shall stay correct if the device sleeps, the tab is in the
  background, or the page is reloaded (time is based on recorded start times, not a ticking counter).
- **FR8.4** The owner shall be able to pause, resume, or stop a session.

Acceptance criteria:
- Given a running focus interval, When I lock my phone for 10 minutes and come back, Then the timer shows about 10 minutes less remaining.
- Given I completed 4 focus intervals, When the 4th ends, Then the next break is 15 minutes.

### FR9: End-of-block and end-of-interval notifications

- **FR9.1** When a Pomodoro interval or time block ends, the system shall notify
  the owner with a sound and a system notification, even when the app is in the background. [Q10 A]
- **FR9.2** At the end of a block, the system shall ask whether the item is
  done, needs more time (extend or add a new block today), or should be rescheduled (back to Do Now, or Defer).
- **FR9.3** If notifications are not allowed on a device, the app shall say so and show the prompt the next time it is opened.

Acceptance criteria:
- Given the app is in the background and a block ends, When notifications are allowed, Then I receive a notification within 30 seconds of the end time.
- Given a block ended, When I choose "Needs more time", Then I can extend it or create a new block today.

### FR10: Communication batching reminders

- **FR10.1** The owner shall be able to set daily communication windows,
  defaulting to 10:00, 13:00 and 16:00. [doc §1.4, Q11 B]
- **FR10.2** The system shall send a reminder at the start of each window.
  Windows are reminders only and do not create time blocks or calendar events.

Acceptance criteria:
- Given windows at 10:00, 13:00 and 16:00, When it is 13:00, Then I get a reminder to check messages.

### FR11: Daily verse at planning time

- **FR11.1** When the owner opens today's plan for the first time each day, the
  system shall show one short verse or reminder from a built-in set that includes
  Ephesians 5:15–16, Proverbs 6:6–8 and Matthew 6:34. [doc §3.3, Q12 A]
- **FR11.2** It can be dismissed with one action and is not shown again that day.

### FR12: Weekly summary

- **FR12.1** The system shall show a weekly summary (Monday to Sunday, owner's
  time zone) with: days on which a plan was made, planned blocks, blocks marked
  done, and the current open-list count against the owner's chosen limit. [Q13 C]
- **FR12.2** The owner shall be able to set the open-list limit (a whole number of 1 or more).
- **FR12.3** In a week with no planned blocks, completion shall show "no blocks
  planned", not 0% or an error.

Acceptance criteria:
- Given 5 days planned and 18 of 24 blocks done, When I open the summary, Then it shows 5 planned days and 18/24 blocks done.
- Given no blocks this week, When I open the summary, Then it shows "no blocks planned".

### FR13: Lists and empty states

- **FR13.1** Every list (unsorted, Do Now, Delegated, Deferred, Archive) shall
  show a clear empty state naming the next action.
- **FR13.2** Lists shall have a fixed order: unsorted by capture time; Delegated
  by follow-up date; Deferred by date; ties broken by creation time.
- **FR13.3** The same item open in two tabs or on two devices shall show the same
  data after a refresh; a change made in one place shall never silently overwrite a newer change made in the other.

## Non-Functional Requirements

- **NFR1 Platform**: a responsive web app that works in current Safari (macOS
  and iOS 17+) and Chrome, and can be added to the iPhone home screen. Native apps
  are a possible later phase. [Q1 D]
- **NFR2 Performance**: capturing an item, sorting it, and placing a block each
  show their result in under 300 ms for 95% of actions with up to 2,000 stored items.
- **NFR3 Availability and data safety**: no accepted write is lost; the server's
  data is backed up at least daily and a backup can be restored. Target 99% monthly
  availability for a single-user self-hosted server.
- **NFR4 Security**: all traffic over HTTPS; the Google connection tokens and any
  secrets are stored encrypted and loaded from environment variables or a secrets
  manager, never committed or logged; every input is validated on the server. [project.md Mandated/Forbidden]
- **NFR5 Time**: all times are stored with their offset; "today" is defined in the
  owner's configured time zone and is correct across daylight-saving changes.
- **NFR6 Accessibility**: every function works by keyboard; text contrast meets
  WCAG 2.1 AA; status (for example "not yet synced", overlap) is never shown by color alone.
- **NFR7 Quality**: at least 80% line coverage with tests run in CI before
  merge; lint failures block the merge. [team.md Testing Posture, Code Style]
- **NFR8 Observability**: the server logs every Google sync failure and
  notification-send failure with a timestamp and reason, with no task text or tokens in the logs.

## Constraints

- One owner, solo development; the owner approves all merges and releases. [team.md]
- Self-hosted server; the hosting target and technology stack are chosen in later stages. [Q3 C]
- Dependencies pinned in a lockfile; no secrets in the repository. [project.md Forbidden]
- Lean scope: settings are limited to what the core loop needs (time zone, communication windows, open-list limit, Google connection). [Q14]

## Assumptions

- Background notifications on iPhone require the web app to be added to the home
  screen (iOS 16.4 or later). Reason: this is how iOS delivers web push.
- One Google calendar is chosen for both reading events and writing blocks. Reason: keeps two-way sync simple; a separate "Timely" calendar can be selected if preferred.
- The verse set is a short fixed list shipped with the app; it is not editable in version 1. Reason: Q14 excludes extra settings.
- Delegate names are free text, not linked to contacts. Reason: FU1 defers accounts and sharing.
- Completed and archived items are kept indefinitely in version 1. Reason: single user, small data volume.

## Out of Scope (version 1)

- Accounts for other people and shared or delegated task visibility (later phase). [FU1 C]
- Adjustable Pomodoro lengths (later phase). [Q9]
- Planning beyond today. [Q5 A]
- Communication windows as calendar blocks; Focus-mode or email/Slack integration. [Q11 B]
- Themes, colors and appearance settings; recurring tasks; tags, projects and folders; statistics beyond the weekly summary. [Q14]
- Apple iCloud Calendar sync. [FU2 B]
- Native Mac or iPhone apps. [Q1 D]
- Loom and Focusmate integrations. [doc §2]

## Open Questions

- Which hosting target and technology stack (decided in later design stages).
- Should Timely write to the owner's main Google calendar or a dedicated "Timely" calendar? (Default: owner picks one at connection time.)

## Sources

- `[doc]` `initial_product_idea.md`, "Overwhelm to Execution" handover document (sections §1–§4).
- `[Q<n>]`, `[FU<n>]` `requirements-analysis-questions.md`, answers to Questions 1–14 and Follow-ups 1–2.
- `team.md` and `project.md` affirmed practices from Practices Discovery (`inception/practices-discovery/team-practices.md`).
- No ideation artifacts exist for this workflow (the classic plan skips Ideation); the handover document stands in as the originating intent.
