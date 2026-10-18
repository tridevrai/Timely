# Business Rules: U1 Core loop slice

Every rule is enforced by the server (ADR-009); the web app may check the same
rule early for fast feedback but never instead of the server. "Today" always
means the current date in the owner's time zone (`Owner.timeZone`). Answers
from `functional-design-questions.md` are cited as `[Q<n>]`.

```yaml
rules:
  # BR1 Account and access
  - id: BR1.1
    statement: Nothing but the sign-in screen and sign-in request is available without a valid session.
    category: authorization
    applies_to: every page and every /api/v1 path except POST /session
    trigger: any request
    logic: IF the session cookie is missing, unknown, expired or revoked THEN answer 401 unauthenticated with no data, and the web app shows the sign-in screen.
    violation: 401 unauthenticated; page requests show sign-in; no task, plan or account data in the response
    source: FR1.1
  - id: BR1.2
    statement: Correct credentials create a session and land the owner on today's plan.
    category: authorization
    applies_to: POST /session
    trigger: sign-in submit
    logic: IF the lower-cased email matches the Owner and the password verifies against passwordHash THEN create a Session, set the session cookie (Secure, HttpOnly, SameSite=Lax), return the csrfToken, and the web app opens Today.
    violation: see BR1.3
    source: FR1.1, C1
  - id: BR1.3
    statement: A failed sign-in never says which part was wrong.
    category: authorization
    applies_to: POST /session
    trigger: sign-in submit with wrong email, wrong password, or both
    logic: IF the credentials do not verify THEN answer 401 with the message "Sign-in failed" whatever the cause, taking the same time whether or not the email exists.
    violation: 401 "Sign-in failed"; no data shown
    source: FR1.1, US1.1
  - id: BR1.4
    statement: Exactly one owner exists, created only by the server-side create-owner command.
    category: constraint
    applies_to: Owner
    trigger: running create-owner
    logic: IF an Owner already exists THEN refuse and change nothing. ELSE require a valid email and an IANA time zone, read the password interactively (never from the command line or a file in the repository), require at least 12 characters, store only its hash. There is no sign-up screen or API. [Q1]
    violation: command exits with a non-zero status and a message naming the problem
    source: FR1.2, NFR4
  - id: BR1.5
    statement: Repeated failed sign-ins are slowed down.
    category: policy
    applies_to: POST /session
    trigger: a failed sign-in
    logic: IF 5 failures occur for the same email and client address within 15 minutes THEN refuse further attempts from that key for 15 minutes with 429 rate_limited, even with the right password. A successful sign-in clears the counter.
    violation: 429 rate_limited "Too many attempts. Try again in a few minutes."
    source: C1, NFR4
  - id: BR1.6
    statement: Every state-changing request carries the session's CSRF token.
    category: authorization
    applies_to: every POST, PATCH and DELETE except POST /session
    trigger: state-changing request
    logic: IF the X-CSRF-Token header is missing or differs from Session.csrfToken THEN refuse with 403 csrf_failed and change nothing.
    violation: 403 csrf_failed
    source: C1
  - id: BR1.7
    statement: Every record belongs to the owner, and every read and write is limited to the signed-in owner's records.
    category: authorization
    applies_to: Task, CaptureBatch, TimeBlock, FocusSession, DeviceSubscription, ScheduledAlert
    trigger: any read or write
    logic: IF a requested record's ownerId differs from the session's owner THEN treat it as not found.
    violation: 404 not_found
    source: FR1.2

  # BR2 Capture
  - id: BR2.1
    statement: Each line with visible text becomes exactly one unsorted task; blank lines are ignored.
    category: calculation
    applies_to: POST /tasks/capture
    trigger: capture submit
    logic: Split the submitted lines; trim surrounding whitespace from each; IF the trimmed line is empty THEN skip it ELSE create one Task in state unsorted, keeping the line's position. IF no line remains THEN create nothing and answer 422 "Nothing to add".
    violation: 422 validation_failed for an all-blank submit
    source: FR2.1, FR2.2
  - id: BR2.2
    statement: Capture size limits.
    category: validation
    applies_to: POST /tasks/capture
    trigger: capture submit
    logic: IF more than 200 lines are submitted, or any trimmed line is longer than 2,000 characters (counted as visible characters) THEN refuse the whole submit with the line numbers that break the limit; ELSE store each line whole, never truncated.
    violation: 422 validation_failed naming the offending lines; typed text stays in the box
    source: FR2.1, US2.1
  - id: BR2.3
    statement: Task text is data, never markup.
    category: policy
    applies_to: Task.text everywhere it is shown, including notification titles
    trigger: any display
    logic: Store the text as given and always render it as plain text (escaped for its destination), so emoji show as emoji and markup such as `<script>` shows literally and never runs.
    violation: not applicable; enforced by output encoding
    source: FR2 acceptance criteria, NFR4
  - id: BR2.4
    statement: Repeating a capture submit returns the first result and creates nothing new.
    category: constraint
    applies_to: POST /tasks/capture
    trigger: submit with an Idempotency-Key already used by this owner
    logic: IF a CaptureBatch with this key exists and its contentDigest matches THEN return its original tasks with 201 and create nothing. IF the key exists with a different contentDigest THEN refuse with 422 (a key cannot be reused for other content). The key is required.
    violation: 422 validation_failed for a missing or reused key
    source: FR2.4
  - id: BR2.5
    statement: A capture is reported saved only after it is durably stored.
    category: constraint
    applies_to: POST /tasks/capture
    trigger: capture submit
    logic: Create the CaptureBatch and all its Tasks in one transaction; answer 201 only after the commit. Any failure rolls back the whole batch.
    violation: 500 internal; nothing from the batch exists
    source: FR2.3, NFR3
  - id: BR2.6
    statement: An unsaved capture keeps the typed text and says it was not saved.
    category: policy
    applies_to: Capture screen
    trigger: capture submit fails, or no response within 10 seconds
    logic: IF the response is an error or does not arrive within 10 seconds THEN keep the typed text in the box, show "Not saved. Your text is still here." with Retry, and retry with the same Idempotency-Key so a request that did arrive is not duplicated.
    violation: not applicable
    source: FR2.3, C1 timeouts

  # BR3 2-minute rule
  - id: BR3.1
    statement: Only an unsorted task can be started as a 2-minute task, and it needs no category.
    category: validation
    applies_to: POST /focus-sessions with mode twoMinute
    trigger: start 2-minute
    logic: IF the task is not unsorted THEN refuse. ELSE start a twoMinute FocusSession with no blockId; the task keeps state unsorted while the countdown runs.
    violation: 422 validation_failed "Only unsorted items can be done as 2-minute tasks."
    source: FR3.1, FR3.4
  - id: BR3.2
    statement: The 2-minute countdown lasts exactly 120 seconds from its recorded start.
    category: calculation
    applies_to: FocusSession mode twoMinute
    trigger: any read of the session
    logic: remaining = 120 s - (now - startedAt - pausedMs), never below 0. At 0 the countdown has finished (endReason countdownFinished) and asks "Done?".
    violation: not applicable
    source: FR3.2
  - id: BR3.3
    statement: Answering Done completes the task.
    category: policy
    applies_to: POST /focus-sessions/{id}/answer with done true
    trigger: Done answer after the countdown finished
    logic: Set answer done; complete the Task (state completed, completedAt now, version + 1) and emit TaskCompleted, all in one transaction.
    violation: 422 if the session is not a finished, unanswered twoMinute session
    source: FR3.3
  - id: BR3.4
    statement: Answering Not yet leaves the task unsorted and unchanged.
    category: policy
    applies_to: POST /focus-sessions/{id}/answer with done false
    trigger: Not yet answer
    logic: Set answer notYet; the Task is not modified (same text, same version, still unsorted).
    violation: 422 as for BR3.3
    source: FR3.3
  - id: BR3.5
    statement: An unanswered 2-minute countdown asks again.
    category: policy
    applies_to: web app start-up and Sort screen
    trigger: opening the app while a finished twoMinute session has no answer
    logic: IF such a session exists THEN show the "Done?" question again before anything else on the Sort screen; the task stays unsorted until answered.
    violation: not applicable
    source: FR3.2, corner checklist (killed mid-work)

  # BR4 Sort and lists
  - id: BR4.1
    statement: A task has exactly one state at any time.
    category: constraint
    applies_to: Task
    trigger: every change
    logic: A change replaces the state; there is no multi-category task. Lists are filtered by state, so a task appears in exactly one list.
    violation: not applicable; enforced by the data model
    source: FR4.1
  - id: BR4.2
    statement: An unsorted task can be sorted into Do Now.
    category: policy
    applies_to: PATCH /tasks/{id} with state doNow
    trigger: Do Now chosen on the sort card
    logic: IF the task is unsorted THEN set state doNow, version + 1, emit TaskCategoryChanged (from unsorted, to doNow). The task leaves the unsorted list and appears in Do Now.
    violation: 422 validation_failed if the task is not unsorted
    source: FR4.1, FR4.2
  - id: BR4.3
    statement: Only the slice's state changes are open in U1.
    category: constraint
    applies_to: Task state changes
    trigger: any requested change
    logic: Allowed in U1 - unsorted to doNow (BR4.2), unsorted to completed (BR3.3), doNow to completed (BR5.10, POST /tasks/{id}/complete). Any other change (delegate, defer, drop, restore, back to unsorted) is refused until U2 adds it.
    violation: 422 validation_failed "Not available yet."
    source: unit-of-work.md U1 boundaries
  - id: BR4.4
    statement: A change made from stale data never overwrites a newer one.
    category: constraint
    applies_to: PATCH /tasks/{id}, PATCH /blocks/{id}
    trigger: update with If-Match
    logic: IF If-Match differs from the stored version THEN refuse with 409 conflict and the current record; ELSE apply and increment version. In U1 the web app reloads the record and shows "This changed on another device" (the full choose-a-version prompt is U3).
    violation: 409 conflict with current
    source: FR13.3, ADR-009, C1
  - id: BR4.5
    statement: Lists have a fixed order.
    category: calculation
    applies_to: GET /tasks
    trigger: list read
    logic: unsorted and doNow lists are ordered by capturedAt, then capturePosition, then taskId.
    violation: not applicable
    source: FR13.2
  - id: BR4.6
    statement: An empty unsorted list points to the brain dump.
    category: policy
    applies_to: Sort screen
    trigger: no unsorted tasks
    logic: Show "Inbox clear. Nothing to sort." with a button to open Capture. Loading, empty and error are three different views.
    violation: not applicable
    source: FR13.1, US4.1

  # BR5 Planning today
  - id: BR5.1
    statement: Only a Do Now task can be time-blocked.
    category: validation
    applies_to: POST /blocks
    trigger: create block
    logic: IF Tasks.isDoNow(ownerId, taskId) is false THEN refuse.
    violation: 422 validation_failed "Only Do Now items can be scheduled."
    source: FR4.2, FR6.2
  - id: BR5.2
    statement: Only today can be planned.
    category: validation
    applies_to: POST /blocks, GET /today
    trigger: create block; open Today
    logic: The block's localDate is set by the server to today; IF startsAt falls on any other local date THEN refuse. The timeline offers no other day.
    violation: 422 validation_failed "Only today can be planned."
    source: FR6.1
  - id: BR5.3
    statement: A block ends after it starts.
    category: validation
    applies_to: POST /blocks
    trigger: create block
    logic: IF endsAt <= startsAt THEN refuse and save nothing.
    violation: 422 validation_failed on field endsAt "End must be after start."
    source: FR6.2
  - id: BR5.4
    statement: A block ends today.
    category: validation
    applies_to: POST /blocks
    trigger: create block
    logic: IF endsAt is later than the end of today (local midnight at the start of tomorrow) THEN refuse. A block ending exactly at midnight is allowed.
    violation: 422 validation_failed on field endsAt "Blocks must end today."
    source: FR6.2
  - id: BR5.5
    statement: A block cannot lie wholly in the past.
    category: validation
    applies_to: POST /blocks
    trigger: create block
    logic: IF endsAt <= now THEN refuse. A block may start before now when it ends after now. [Q4]
    violation: 422 validation_failed on field endsAt "This time has already passed."
    source: FR6.1, [Q4]
  - id: BR5.6
    statement: Block times sit on 5-minute marks and a block lasts at least 15 minutes.
    category: validation
    applies_to: POST /blocks, PATCH /blocks/{id}
    trigger: create or move block
    logic: IF startsAt or endsAt is not on a 5-minute mark of local time, or endsAt - startsAt < 15 minutes THEN refuse. Dragging on the Mac snaps to 15 minutes. [Q5]
    violation: 422 validation_failed naming the field
    source: FR6.2, interaction-spec, [Q5]
  - id: BR5.7
    statement: Times are stored as instants with their offset and shown in the owner's time zone.
    category: calculation
    applies_to: TimeBlock, Today timeline
    trigger: create and display
    logic: The client sends local times with offset; the server converts them with Owner.timeZone, stores instants with offset, and the timeline renders them in Owner.timeZone, including on the day of a daylight-saving change.
    violation: 422 validation_failed for a time without offset
    source: NFR5, FR6.1
  - id: BR5.8
    statement: Every planned block has exactly one block-end alert.
    category: policy
    applies_to: TimeBlock
    trigger: BlockCreated, BlockChanged, BlockRemoved
    logic: On BlockCreated or BlockChanged with status planned, schedule (or replace) the alert with sourceKey "block:<blockId>:end" at endsAt, kind blockEnd, title "Time's up: <task text, plain, shortened to 100 characters>". On BlockRemoved or status done, cancel it.
    violation: not applicable
    source: FR9.1, ADR-005, C2
  - id: BR5.9
    statement: The Time's up sheet offers Done or Not yet in this slice.
    category: policy
    applies_to: POST /blocks/{id}/end-decision (U1 subset - done and notYet)
    trigger: the owner answers the Time's up sheet for a block whose endsAt has passed
    logic: IF Done THEN mark the block done and complete the task (BR5.10). IF Not yet THEN leave the block planned and the task doNow. Either answer sets endAcknowledgedAt so the sheet is not shown again. Extend, new block and reschedule arrive in U3. [Q2]
    violation: 422 if the block has not ended or was already acknowledged
    source: FR9.2, [Q2]
  - id: BR5.10
    statement: Completing a task marks its planned blocks done.
    category: policy
    applies_to: TimeBlock
    trigger: TaskCompleted
    logic: Mark every planned block of the task done (version + 1), which cancels their block-end alerts (BR5.8). Applying the same TaskCompleted event twice changes nothing more.
    violation: not applicable
    source: components.md Planning, C2

  # BR6 Focus
  - id: BR6.1
    statement: A Pomodoro starts from a planned block of today and works on that block's task.
    category: validation
    applies_to: POST /focus-sessions with mode pomodoro
    trigger: Focus started from a block
    logic: IF the block is not the owner's, is not planned, is not today, or its endsAt has passed THEN refuse. ELSE create a pomodoro FocusSession with the block's taskId and blockId, startedAt now, phase focus.
    violation: 422 validation_failed "This block has already ended."
    source: FR8.1
  - id: BR6.2
    statement: The Pomodoro cycle is 25 minutes focus, 5 minutes break, and 15 minutes after every 4th focus interval.
    category: calculation
    applies_to: FocusSession mode pomodoro
    trigger: any read of the session
    logic: The phases repeat focus 25, break 5, focus 25, break 5, focus 25, break 5, focus 25, longBreak 15, then start again. Lengths are fixed in version 1.
    violation: not applicable
    source: FR8.2
  - id: BR6.3
    statement: Remaining time is computed from the recorded start, never counted down.
    category: calculation
    applies_to: FocusSession
    trigger: any read of the session
    logic: elapsed = now - startedAt - pausedMs. The current phase, completedIntervals and phaseEndsAt follow from elapsed and BR6.2 (or BR3.2). Server time is the reference; the client corrects its display with serverTime.
    violation: not applicable
    source: FR8.3
  - id: BR6.4
    statement: Phase ends are announced.
    category: policy
    applies_to: FocusSession mode pomodoro and twoMinute
    trigger: session start and each phase end
    logic: Keep exactly one pending alert with sourceKey "focus:<sessionId>:phase" at the current phaseEndsAt, kind intervalEnd ("Break time" after focus, "Back to focus" after a break, "2 minutes are up" for twoMinute). When it fires, schedule the next phase's alert. Stopping cancels it.
    violation: not applicable
    source: FR9.1
  - id: BR6.5
    statement: A Pomodoro stops when its block ends, with one alert.
    category: policy
    applies_to: FocusSession mode pomodoro
    trigger: the block's endsAt arrives during the session
    logic: End the session at the block's endsAt (endReason blockEnded). IF a phase end would fall at or after the block's endsAt THEN schedule no phase alert for it; the block-end alert (BR5.8) is the only alert, and the web app opens the Time's up sheet. [Q3]
    violation: not applicable
    source: FR8.1, FR9.1, [Q3]
  - id: BR6.6
    statement: Only one countdown runs at a time.
    category: constraint
    applies_to: POST /focus-sessions
    trigger: start a countdown while another session has no endedAt
    logic: IF another session is running THEN refuse with 409 conflict naming the running session; the web app asks "Stop the current countdown and start this one?" and, on yes, stops it (endReason replaced) and starts the new one. [Q6]
    violation: 409 conflict with current set to the running session
    source: ADR-002, [Q6]
  - id: BR6.7
    statement: Stopping a session is safe to repeat.
    category: constraint
    applies_to: POST /focus-sessions/{id}/stop
    trigger: stop
    logic: IF the session already ended THEN return it unchanged. ELSE set endedAt now, endReason stopped, and cancel its phase alert. The task and block are not changed.
    violation: not applicable
    source: US6.3 (stop part in U1)

  # BR7 Notifications
  - id: BR7.1
    statement: A device registers to receive notifications.
    category: validation
    applies_to: POST /push-subscriptions
    trigger: the owner allows notifications on a device
    logic: Accept only an https endpoint with both public keys; IF the endpoint is already registered THEN update it instead of adding a second one.
    violation: 422 validation_failed
    source: FR9.1, C5
  - id: BR7.2
    statement: A due alert reaches every registered device within 30 seconds.
    category: policy
    applies_to: ScheduledAlert
    trigger: fireAt reached
    logic: The in-app scheduler checks at least every 10 seconds and sends each pending alert whose fireAt has passed to every DeviceSubscription of the owner, marked to play a sound, then sets status sent. An alert is sent at most once even if the scheduler runs twice. The web app also plays a sound when it is open.
    violation: not applicable
    source: FR9.1, C5
  - id: BR7.3
    statement: Push delivery failures are handled and logged without task text.
    category: policy
    applies_to: ScheduledAlert delivery
    trigger: push service answers with an error
    logic: 404 or 410 removes that DeviceSubscription. 429 or 5xx retries up to 3 times within 60 seconds, then marks the alert failed. Every failure is logged with time, status code and alertId only.
    violation: not applicable
    source: C5, NFR8
  - id: BR7.4
    statement: The app says when notifications are off and how to turn them on.
    category: policy
    applies_to: web app start-up, Today
    trigger: the device has not granted notification permission, or no subscription exists
    logic: Show a note "Notifications are off on this device" with the steps for this browser (on iPhone - add to home screen, then allow notifications). The note never relies on color alone.
    violation: not applicable
    source: FR9.3
  - id: BR7.5
    statement: Missed ends are shown when the app is next opened.
    category: policy
    applies_to: web app start-up, Today
    trigger: opening the app
    logic: IF a block of today has endsAt in the past, status planned and no endAcknowledgedAt THEN show its Time's up sheet (BR5.9), one block at a time, oldest first. This happens whether or not a notification was delivered.
    violation: not applicable
    source: FR9.3, US7.1

  # BR8 App shell
  - id: BR8.1
    statement: Configuration and secrets come from the environment only.
    category: policy
    applies_to: server start-up
    trigger: start
    logic: Read the database location, session secret and VAPID keys from environment variables; IF any is missing or malformed THEN refuse to start and name the missing key (never its value). Secrets are never logged.
    violation: start-up error naming the key
    source: NFR4, project.md Forbidden
  - id: BR8.2
    statement: Every change and its events are saved together.
    category: constraint
    applies_to: OutboxEvent
    trigger: any change that emits an event
    logic: Write the OutboxEvent in the same transaction as the change. Deliver in commit order per aggregate; subscribers ignore an eventId they already handled; a failing handler is retried with capped backoff and logged after 5 failures.
    violation: not applicable
    source: C2, ADR-010
```

## Rules summary

| ID | Rule | Category |
|---|---|---|
| BR1.1 | No data without a valid session | authorization |
| BR1.2 | Correct credentials open a session and land on Today | authorization |
| BR1.3 | Failed sign-in says only "Sign-in failed" | authorization |
| BR1.4 | One owner, created only by the create-owner command | constraint |
| BR1.5 | 5 failed sign-ins in 15 minutes block that key for 15 minutes | policy |
| BR1.6 | State-changing requests need the CSRF token | authorization |
| BR1.7 | Every read and write is limited to the owner's records | authorization |
| BR2.1 | Each non-blank line becomes one unsorted task | calculation |
| BR2.2 | At most 200 lines, 2,000 characters each, never truncated | validation |
| BR2.3 | Task text is always shown as plain text | policy |
| BR2.4 | A repeated capture key creates nothing new | constraint |
| BR2.5 | Saved means committed, all or nothing | constraint |
| BR2.6 | A failed capture keeps the text and retries with the same key | policy |
| BR3.1 | Only unsorted tasks can be 2-minute tasks; no category needed | validation |
| BR3.2 | The countdown is 120 seconds from its recorded start | calculation |
| BR3.3 | Done completes the task | policy |
| BR3.4 | Not yet leaves the task unsorted and unchanged | policy |
| BR3.5 | An unanswered countdown asks again on next open | policy |
| BR4.1 | One state per task | constraint |
| BR4.2 | Unsorted to Do Now | policy |
| BR4.3 | Only the slice's state changes are open in U1 | constraint |
| BR4.4 | Stale versions are refused with 409 | constraint |
| BR4.5 | Fixed list order | calculation |
| BR4.6 | Empty unsorted list points to Capture | policy |
| BR5.1 | Only Do Now tasks can be blocked | validation |
| BR5.2 | Only today can be planned | validation |
| BR5.3 | End after start | validation |
| BR5.4 | End no later than midnight | validation |
| BR5.5 | End after now | validation |
| BR5.6 | 5-minute marks, at least 15 minutes | validation |
| BR5.7 | Instants with offset, shown in the owner's time zone | calculation |
| BR5.8 | One block-end alert per planned block | policy |
| BR5.9 | Time's up: Done or Not yet | policy |
| BR5.10 | Completed task marks its blocks done | policy |
| BR6.1 | Pomodoro starts from a planned, not-yet-ended block of today | validation |
| BR6.2 | 25/5, long break 15 after every 4th focus | calculation |
| BR6.3 | Remaining time from recorded start | calculation |
| BR6.4 | One pending phase alert per session | policy |
| BR6.5 | Pomodoro stops at block end, one alert | policy |
| BR6.6 | One countdown at a time | constraint |
| BR6.7 | Stop is safe to repeat | constraint |
| BR7.1 | Device registration | validation |
| BR7.2 | Due alerts sent within 30 seconds, once | policy |
| BR7.3 | Push failures handled and logged without task text | policy |
| BR7.4 | "Notifications are off" note | policy |
| BR7.5 | Missed block ends shown on next open | policy |
| BR8.1 | Configuration and secrets from the environment | policy |
| BR8.2 | Changes and their events saved together | constraint |
