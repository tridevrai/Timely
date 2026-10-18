# Contract Summary: Timely

Inputs: `units-generation/unit-of-work.md` and `unit-of-work-dependency.md`
(5 units in one app), `domain-design/components.md` (9 components, ADR-001 to
ADR-010), `requirements.md`, and the answers in `contract-design-questions.md`
(all option A).

Timely is a single deployable app [units Q1], so the unit DAG edges are
in-process. The contracts that must be pinned before Code Generation are:

1. **C1: HTTP API**, the external boundary between the browser (Mac and iPhone) and the server. [Q1, Q2, Q3, Q5, Q7]
2. **C2: Internal events**, the in-process "something changed" events, stored with each change and delivered afterwards. [Q6, ADR-010]
3. **C3: Module interfaces**, the calls between building blocks that cross unit edges.
4. **C4: Google Calendar usage**, how CalendarSync uses Google's fixed API. [Q4, ADR-004]
5. **C5: Web push usage**, how Notifications delivers alerts.

## Contracts

| # | Provider Unit | Consumer | Mechanism | Owner |
|---|---|---|---|---|
| C1 | U1 (extended by U2–U5) | External: the web app in the browser | Synchronous JSON over HTTPS, `/api/v1` | U1 owns the base, the conventions and the error shape; each unit owns the paths it adds |
| C2 | U1 (Tasks, Planning, Account) | U2 (Planning reacting to Tasks), U4 (CalendarSync), U5 (Notifications) | Async in-process events via a transactional outbox | Emitting component's unit (U1) |
| C3 | U1 (Tasks, Planning, Account, Notifications, Focus) | U2, U3, U4, U5 | Synchronous in-process module interfaces | Providing component's unit (U1) |
| C4 | External: Google Calendar API | U4 (CalendarSync) | Synchronous HTTPS with OAuth 2.0; incremental sync every 2 minutes | Google (format); U4 (our usage) |
| C5 | External: browser push services | U1 (Notifications) | Web Push protocol with VAPID keys | Standard (format); U1 (our usage) |

---

## C1: HTTP API (browser ↔ server)

**Conventions** (apply to every path):
- **Authentication**: a session cookie (`Secure`, `HttpOnly`, `SameSite=Lax`) set at sign-in. Every state-changing request also sends the `X-CSRF-Token` header, matching the token returned at sign-in. Missing or invalid session → `401`. [Q2]
- **Versioning**: every path starts with `/api/v1`. The web app ships with the server, so both change together. Additive changes (new fields, new paths) are always allowed, and clients ignore fields they don't know. A breaking change adds `/api/v2`, but only if an outside client ever exists. [Q7]
- **Concurrency**: `Task` and `TimeBlock` carry an integer `version`. Updates send `If-Match: <version>`. A stale version → `409 conflict` with the current resource in `current`, and the app asks the owner which version to keep. [Q5, ADR-009]
- **Idempotency**: `POST /tasks/capture` requires an `Idempotency-Key` header. Repeating it returns the original result and creates nothing. [FR2.4]
- **Freshness**: the app re-fetches when it becomes visible and every 60 seconds while open. Responses carry `serverTime` so the app can show "last updated". [Q3]
- **Times**: instants are RFC 3339 with an offset; `localDate` is `YYYY-MM-DD` in the owner's time zone. [NFR5]
- **Validation**: the server validates every input (project.md Mandated rule); field errors come back with the field name.
- **Timeouts**: the app treats no response within 10 seconds as offline, keeps the typed input, and offers to retry. The server answers within NFR2's targets.

**Error body** (every 4xx and 5xx):

```yaml
Error:
  type: object
  required: [code, message]
  properties:
    code: { type: string, enum: [unauthenticated, csrf_failed, validation_failed, not_found, conflict, rate_limited, google_unavailable, internal] }
    message: { type: string, description: plain-language sentence safe to show the owner }
    fields:
      type: array
      items: { type: object, properties: { field: { type: string }, problem: { type: string } } }
    current: { description: on 409 only, the current server copy of the resource }
```

```yaml
openapi: 3.1.0
info: { title: Timely API, version: "1" }
servers: [{ url: /api/v1 }]
components:
  securitySchemes:
    session: { type: apiKey, in: cookie, name: timely_session }
  schemas:
    Task:
      type: object
      required: [id, text, state, version, capturedAt]
      properties:
        id: { type: string }
        text: { type: string, maxLength: 2000 }
        state: { type: string, enum: [unsorted, doNow, delegated, deferred, archived, completed] }
        nextStep: { type: [string, "null"], maxLength: 500 }
        delegateName: { type: [string, "null"], maxLength: 100 }
        followUpDate: { type: [string, "null"], format: date }
        deferUntil: { type: [string, "null"], format: date }
        capturedAt: { type: string, format: date-time }
        completedAt: { type: [string, "null"], format: date-time }
        version: { type: integer }
    TimeBlock:
      type: object
      required: [id, taskId, localDate, startsAt, endsAt, status, version]
      properties:
        id: { type: string }
        taskId: { type: string }
        localDate: { type: string, format: date }
        startsAt: { type: string, format: date-time }
        endsAt: { type: string, format: date-time }
        status: { type: string, enum: [planned, done] }
        overlaps: { type: array, items: { type: string }, description: ids of blocks it overlaps }
        syncState: { type: string, enum: [notConnected, synced, pending, failed] }
        version: { type: integer }
    CalendarEvent:
      type: object
      properties:
        title: { type: string }
        startsAt: { type: string, format: date-time }
        endsAt: { type: string, format: date-time }
        allDay: { type: boolean }
    FocusSession:
      type: object
      required: [id, mode, phase, startedAt, pausedMs, phaseEndsAt]
      properties:
        id: { type: string }
        taskId: { type: string }
        blockId: { type: [string, "null"] }
        mode: { type: string, enum: [pomodoro, twoMinute, justStart] }
        phase: { type: string, enum: [focus, break, longBreak, ended] }
        completedIntervals: { type: integer }
        startedAt: { type: string, format: date-time }
        pausedAt: { type: [string, "null"], format: date-time }
        pausedMs: { type: integer }
        phaseEndsAt: { type: [string, "null"], format: date-time, description: null while paused }
    Preferences:
      type: object
      properties:
        timeZone: { type: string, description: IANA name }
        messageWindows: { type: array, items: { type: string, pattern: "^[0-2][0-9]:[0-5][0-9]$" } }
        openListLimit: { type: integer, minimum: 1 }
security: [{ session: [] }]
paths:
  /session:
    post:
      summary: Sign in (U1)
      security: []
      requestBody: { content: { application/json: { schema: { type: object, required: [email, password], properties: { email: { type: string }, password: { type: string } } } } } }
      responses:
        "200": { description: signed in; sets cookie; body has csrfToken }
        "401": { description: "generic 'Sign-in failed' message" }
        "429": { description: too many attempts }
    delete:
      summary: Sign out (U1)
      responses: { "204": { description: signed out } }
  /me/preferences:
    get: { summary: Read preferences (U1), responses: { "200": { description: Preferences } } }
    patch: { summary: Update preferences (U1 time zone; U5 windows and limit), responses: { "200": { description: Preferences }, "422": { description: validation_failed } } }
  /tasks/capture:
    post:
      summary: Create unsorted tasks from lines (U1)
      parameters: [{ name: Idempotency-Key, in: header, required: true, schema: { type: string } }]
      requestBody: { content: { application/json: { schema: { type: object, required: [lines], properties: { lines: { type: array, maxItems: 200, items: { type: string } } } } } } }
      responses: { "201": { description: created Task list (blank lines ignored) } }
  /tasks:
    get:
      summary: List tasks by state, in the fixed order of FR13.2 (U1; U2 adds states)
      parameters: [{ name: state, in: query, required: true, schema: { type: string } }]
      responses: { "200": { description: Task list and serverTime } }
  /tasks/{id}:
    patch:
      summary: "Change state and details: sort, delegate, defer, drop, restore, next step (U1 doNow; U2 the rest)"
      parameters: [{ name: If-Match, in: header, required: true, schema: { type: integer } }]
      responses:
        "200": { description: updated Task }
        "409": { description: conflict with current Task }
        "422": { description: "validation_failed, for example a defer date not in the future" }
  /tasks/{id}/complete:
    post: { summary: Mark completed (U1), responses: { "200": { description: Task } } }
  /follow-ups/today:
    get: { summary: Delegated tasks with followUpDate today (U2), responses: { "200": { description: Task list } } }
  /tasks/{id}/follow-up:
    post:
      summary: Follow-up action done | pushDate | takeBack (U2)
      responses: { "200": { description: Task }, "409": { description: conflict } }
  /today:
    get:
      summary: "Today's plan: blocks, Google events, sync status, verse, follow-ups count (U1; U4 and U5 add fields)"
      responses: { "200": { description: "{ localDate, blocks: TimeBlock[], events: CalendarEvent[], sync: { connected, lastSyncedAt, retrying }, serverTime }" } }
  /blocks:
    post:
      summary: Create a block for a doNow task today (U1)
      responses:
        "201": { description: TimeBlock }
        "422": { description: "end before start, not today, or task not doNow" }
  /blocks/{id}:
    patch:
      summary: Move or resize (U1, U3)
      parameters: [{ name: If-Match, in: header, required: true, schema: { type: integer } }]
      responses: { "200": { description: TimeBlock }, "409": { description: conflict } }
    delete:
      summary: Remove a block (U1)
      responses: { "204": { description: removed } }
  /blocks/{id}/end-decision:
    post:
      summary: "done | extend {minutes} | newBlock {startsAt, endsAt} | backToDoNow | defer {date} (U3)"
      responses: { "200": { description: updated TimeBlock and Task } }
  /focus-sessions:
    post:
      summary: Start a session { mode, taskId, blockId? } (U1; U3 adds justStart)
      responses: { "201": { description: FocusSession } }
  /focus-sessions/current:
    get: { summary: The running session or 404 (U1), responses: { "200": { description: FocusSession } } }
  /focus-sessions/{id}/{action}:
    post:
      summary: "pause | resume | stop | answer {done: boolean} (U1 stop and answer; U3 pause and resume)"
      responses: { "200": { description: FocusSession } }
  /push-subscriptions:
    post: { summary: Register this device for notifications (U1), responses: { "201": { description: registered } } }
    get: { summary: Whether any device can receive notifications (U1), responses: { "200": { description: "{ enabled: boolean }" } } }
  /calendar/connect:
    post: { summary: Start Google consent; returns the redirect URL (U4), responses: { "200": { description: "{ url }" } } }
  /calendar/callback:
    get: { summary: Google OAuth callback; validates state, stores encrypted tokens (U4), security: [], responses: { "302": { description: back to Settings } } }
  /calendar/connection:
    get: { summary: "Connection status and chosen calendar (U4)", responses: { "200": { description: "{ status, calendarId, lastSyncedAt }" } } }
    patch: { summary: Choose the calendar (U4), responses: { "200": { description: updated } } }
    delete: { summary: Disconnect (U4), responses: { "204": { description: disconnected } } }
  /calendar/notices:
    get: { summary: Unseen conflict notices (U4), responses: { "200": { description: "[{ id, blockId, keptSide, replacedSummary }]" } } }
  /calendar/notices/{id}/seen:
    post: { summary: Mark a notice seen (U4), responses: { "204": { description: ok } } }
  /verse/today:
    get: { summary: "Today's verse, or 204 if dismissed (U5)", responses: { "200": { description: "{ reference, text }" } } }
  /verse/today/dismiss:
    post: { summary: Dismiss for today (U5), responses: { "204": { description: ok } } }
  /summary/week:
    get:
      summary: Weekly summary for the week containing ?date (U5)
      responses: { "200": { description: "{ weekStart, daysPlanned, blocksPlanned, blocksDone, openCount, openLimit }" } }
```

## C2: Internal events (transactional outbox)

Every event is written to an `outbox` table **in the same database transaction**
as the change that caused it, then delivered in-process to subscribers by a
worker that runs inside the app. Delivery is at-least-once, in commit order per
aggregate. Subscribers are idempotent by `eventId`. A failed handler is retried
with capped backoff and is visible in the logs after 5 failures. [Q6, ADR-010]
Payloads carry IDs and times only, never task text.

```yaml
asyncapi: 3.0.0
info: { title: Timely internal events, version: "1" }
channels:
  tasks:
    messages:
      TaskCategoryChanged:
        payload: { eventId: string, ownerId: string, taskId: string, from: string, to: string, occurredAt: date-time }
      TaskCompleted:
        payload: { eventId: string, ownerId: string, taskId: string, occurredAt: date-time }
  blocks:
    messages:
      BlockCreated:
        payload: { eventId: string, ownerId: string, blockId: string, version: integer, startsAt: date-time, endsAt: date-time, occurredAt: date-time }
      BlockChanged:
        payload: { eventId: string, ownerId: string, blockId: string, version: integer, startsAt: date-time, endsAt: date-time, status: string, origin: "timely | google", occurredAt: date-time }
      BlockRemoved:
        payload: { eventId: string, ownerId: string, blockId: string, origin: "timely | google", occurredAt: date-time }
  account:
    messages:
      PreferencesChanged:
        payload: { eventId: string, ownerId: string, changed: "[timeZone | messageWindows | openListLimit]", occurredAt: date-time }
subscriptions:
  - { event: TaskCategoryChanged, subscriber: Planning, action: "remove the task's planned blocks today when it leaves doNow" }
  - { event: TaskCompleted, subscriber: Planning, action: mark the task's open blocks done }
  - { event: BlockCreated, subscriber: CalendarSync, action: create the Google event }
  - { event: BlockChanged, subscriber: CalendarSync, action: "update the Google event; ignored when origin is google" }
  - { event: BlockRemoved, subscriber: CalendarSync, action: "delete the Google event; ignored when origin is google" }
  - { event: BlockCreated / BlockChanged / BlockRemoved, subscriber: Planning (self), action: schedule, reschedule or cancel the block-end alert }
  - { event: PreferencesChanged, subscriber: Notifications, action: rebuild message-window reminders }
```

The `origin` field stops a Google-side change applied by CalendarSync from being
written straight back to Google (no echo loop).

## C3: Module interfaces (in-process)

Calls between building blocks go through these interfaces only, never another
module's tables. Errors are typed results (`NotFound`, `Conflict`, `Invalid`),
not exceptions crossing the boundary.

```yaml
interfaces:
  Account:
    - getOwnerContext(ownerId) -> { timeZone, messageWindows, openListLimit }
    - today(ownerId) -> localDate
  Tasks:
    - getTask(ownerId, taskId) -> Task | NotFound
    - isDoNow(ownerId, taskId) -> boolean
    - complete(ownerId, taskId) -> Task                       # Focus after a 2-minute Done; Planning end decision
    - moveTo(ownerId, taskId, state, details, expectedVersion) -> Task | Conflict | Invalid   # U3 end decision reschedule
    - countOpen(ownerId) -> integer                            # Insights
  Planning:
    - getBlock(ownerId, blockId) -> TimeBlock | NotFound        # Focus
    - applyExternalChange(ownerId, blockId, { startsAt, endsAt } | removed, googleUpdatedAt) -> TimeBlock | Removed   # CalendarSync, emits events with origin google
    - blocksForWeek(ownerId, weekStart) -> [{ localDate, planned, done }]   # Insights
  Notifications:
    - schedule(ownerId, sourceKey, kind, fireAt, title) -> alertId   # replaces any alert with the same sourceKey
    - cancel(ownerId, sourceKey)
```

This answers the domain-design review point: CalendarSync both **subscribes**
to block events (C2) and **calls** `Planning.applyExternalChange` (C3) for
Google-side changes.

## C4: Google Calendar usage (U4)

```yaml
provider: Google Calendar API v3
auth:
  flow: OAuth 2.0 authorization code with PKCE, offline access
  scope: https://www.googleapis.com/auth/calendar.events     # events only; no calendar settings
  tokens: refresh token stored encrypted (key from environment); access tokens refreshed on expiry
reads:
  today: events.list(calendarId, timeMin=start of today, timeMax=end of today, singleEvents=true)
  changes: events.list(calendarId, syncToken) every 2 minutes and on opening Today; 410 Gone -> full resync of today
writes:
  create: events.insert with extendedProperties.private.timelyBlockId=<blockId>   # lets a retry find an already-created event instead of duplicating it
  update: events.patch(eventId)
  delete: events.delete(eventId); 404/410 treated as already deleted
conflicts:
  rule: if both sides changed since lastSyncedBlockVersion, keep the side with the later modification time (block updatedAt vs event updated); record a ConflictNotice
failures:
  timeout: 10 s per call
  retry: 429 and 5xx with exponential backoff and jitter, capped at 5 minutes between tries
  auth_revoked: invalid_grant -> connection status disconnected, notice to the owner, blocks kept
  logging: status code and blockId only, never tokens or task text
```

## C5: Web push usage (Notifications, U1)

```yaml
protocol: Web Push (RFC 8030) with VAPID (RFC 8292); keys from environment variables
payload: { kind: "intervalEnd | blockEnd | messageWindow", title: "short text, no task details beyond the title", url: "/today or /focus" }
delivery: due alerts are sent by the in-app scheduler within 30 seconds of fireAt
failures:
  "404/410": remove the device subscription
  "429/5xx": retry up to 3 times within 60 seconds, then mark failed and log
ios_note: background delivery on iPhone requires the app to be added to the home screen (requirements Assumptions)
```

## Contract ownership rules

- U1 owns the C1 conventions (auth, versioning, errors, concurrency) and every path it introduces. Each later unit owns the paths it adds, which must follow the same conventions.
- The component that emits an event (C2) or provides an interface (C3) owns its shape. Its unit (U1 for all of them today) must keep it backward compatible.
- **Additive changes are always safe**: new optional fields, new paths, new event types. Consumers ignore unknown fields and event types.
- **Breaking changes** (removing or renaming a field, changing a meaning or a status code) need the owner's approval in the unit that makes them. For C1 they ship together with the web app in the same release, because both are one app. For C2, the outbox must be drained before deploying a changed event shape, or both shapes accepted for one release.
- C4 and C5 formats belong to Google and the push standard. Timely's usage is owned by U4 and U1 respectively and isolated in their adapters.

## Open questions

| Contract | Question | Blocks |
|---|---|---|
| C1 | Session lifetime and idle timeout (for example 30 days with sliding renewal?) is an NFR choice | U1 (decided in NFR Requirements) |
| C4 | Write blocks to the owner's main calendar or a dedicated "Timely" calendar? The default is that the owner picks at connect time (requirements Open Questions) | U4 |
| C4 | Clock skew between the server and Google affects "later modification wins"; acceptable tolerance to be set | U4 (NFR Design) |
| C5 | Long-lived VAPID key rotation procedure | U1 (NFR Design) |
