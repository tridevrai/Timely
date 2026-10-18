# Entities: U1 Core loop slice

The data this slice needs, as owned by the components in
`inception/domain-design/components.md`. Attribute names follow the component
catalogue and the HTTP contract (C1). Only the parts the eight slice stories
need are designed here; later units add attributes without changing these.
Technology-agnostic: logical types only.

```yaml
entities:
  - name: Owner
    owned_by: Account
    description: The single person who uses Timely. Exactly one exists in version 1, created by the server-side create-owner command.
    attributes:
      - { name: ownerId, type: identifier, required: true, unique: true }
      - { name: email, type: text, required: true, unique: true, constraints: "valid email form, stored lower-cased, max 254 characters" }
      - { name: passwordHash, type: secret-hash, required: true, constraints: "slow salted password hash; the password itself is never stored or logged" }
      - { name: timeZone, type: text, required: true, constraints: "IANA time zone name, validated against the known list; defines 'today' everywhere" }
      - { name: messageWindows, type: list-of-time-of-day, required: true, default: ["10:00", "13:00", "16:00"], constraints: "used from U5; stored now so the record shape is final" }
      - { name: openListLimit, type: integer, required: false, min: 1, constraints: "used from U5" }
      - { name: createdAt, type: instant, required: true }
      - { name: updatedAt, type: instant, required: true }
    constraints:
      - "At most one Owner record exists in version 1 (BR1.4)."
    relationships:
      - { to: Session, cardinality: "1 to many", direction: "Owner has Sessions" }
      - { to: Task, cardinality: "1 to many", direction: "Owner has Tasks" }
      - { to: DeviceSubscription, cardinality: "1 to many", direction: "Owner has DeviceSubscriptions" }

  - name: Session
    owned_by: Account
    description: A signed-in browser. Identified by the session cookie; carries the CSRF token for state-changing requests.
    attributes:
      - { name: sessionId, type: identifier, required: true, unique: true, constraints: "unguessable random value; only its hash is stored" }
      - { name: ownerId, type: reference, required: true, references: Owner.ownerId }
      - { name: csrfToken, type: secret-token, required: true }
      - { name: createdAt, type: instant, required: true }
      - { name: lastSeenAt, type: instant, required: true }
      - { name: expiresAt, type: instant, required: true, constraints: "lifetime and idle timeout decided in NFR Requirements" }
      - { name: revokedAt, type: instant, required: false, constraints: "set on sign-out" }
    constraints:
      - "A session is valid only while now < expiresAt and revokedAt is empty."
    relationships:
      - { to: Owner, cardinality: "many to 1", direction: "Session belongs to Owner" }

  - name: SignInAttempt
    owned_by: Account
    description: Counter of recent failed sign-ins, used to slow down password guessing (C1 429 response).
    attributes:
      - { name: key, type: text, required: true, unique: true, constraints: "the normalized email plus the client address" }
      - { name: failures, type: integer, required: true, min: 0 }
      - { name: windowStartedAt, type: instant, required: true }
      - { name: blockedUntil, type: instant, required: false }
    relationships: []

  - name: Task
    owned_by: Tasks
    description: One thing the owner captured, from unsorted to done. In this slice it moves unsorted -> doNow -> completed, or unsorted -> completed via the 2-minute rule.
    attributes:
      - { name: taskId, type: identifier, required: true, unique: true }
      - { name: ownerId, type: reference, required: true, references: Owner.ownerId }
      - { name: text, type: text, required: true, min: 1, max: 2000, constraints: "trimmed of surrounding whitespace; stored exactly otherwise; never interpreted as markup" }
      - { name: state, type: enum, required: true, allowed: [unsorted, doNow, delegated, deferred, archived, completed], default: unsorted, constraints: "U1 uses unsorted, doNow and completed only" }
      - { name: nextStep, type: text, required: false, max: 500, constraints: "set from U2" }
      - { name: delegateName, type: text, required: false, max: 100, constraints: "set from U2" }
      - { name: followUpDate, type: local-date, required: false, constraints: "set from U2" }
      - { name: deferUntil, type: local-date, required: false, constraints: "set from U2" }
      - { name: captureBatchKey, type: reference, required: true, references: CaptureBatch.idempotencyKey }
      - { name: capturePosition, type: integer, required: true, min: 0, constraints: "line position inside its capture batch, for a stable order" }
      - { name: capturedAt, type: instant, required: true }
      - { name: completedAt, type: instant, required: false, constraints: "set exactly when state becomes completed" }
      - { name: archivedAt, type: instant, required: false, constraints: "set from U2" }
      - { name: updatedAt, type: instant, required: true }
      - { name: version, type: integer, required: true, min: 1, default: 1, constraints: "incremented on every change (FR13.3, ADR-009)" }
    constraints:
      - "Exactly one state at a time (BR4.1)."
      - "completedAt is present if and only if state is completed."
    relationships:
      - { to: Owner, cardinality: "many to 1", direction: "Task belongs to Owner" }
      - { to: CaptureBatch, cardinality: "many to 1", direction: "Task was created by one CaptureBatch" }
      - { to: TimeBlock, cardinality: "1 to many", direction: "Task is scheduled by TimeBlocks" }
      - { to: FocusSession, cardinality: "1 to many", direction: "Task is worked on in FocusSessions" }

  - name: CaptureBatch
    owned_by: Tasks
    description: One submit of the capture screen. Makes a repeated submit return the original result instead of creating items twice (FR2.4).
    attributes:
      - { name: idempotencyKey, type: text, required: true, unique: true, constraints: "unique per owner; generated by the browser per submit; 8 to 128 characters" }
      - { name: ownerId, type: reference, required: true, references: Owner.ownerId }
      - { name: contentDigest, type: text, required: true, constraints: "digest of the submitted lines, to detect a key reused for different content" }
      - { name: taskIds, type: list-of-reference, required: true, references: Task.taskId }
      - { name: createdAt, type: instant, required: true }
    relationships:
      - { to: Task, cardinality: "1 to many", direction: "CaptureBatch created Tasks" }

  - name: TimeBlock
    owned_by: Planning
    description: A slot on today's timeline for one doNow task.
    attributes:
      - { name: blockId, type: identifier, required: true, unique: true }
      - { name: ownerId, type: reference, required: true, references: Owner.ownerId }
      - { name: taskId, type: reference, required: true, references: Task.taskId }
      - { name: localDate, type: local-date, required: true, constraints: "the owner's today when created" }
      - { name: startsAt, type: instant-with-offset, required: true, constraints: "on a 5-minute mark" }
      - { name: endsAt, type: instant-with-offset, required: true, constraints: "on a 5-minute mark; at least 15 minutes after startsAt; no later than the end of localDate" }
      - { name: status, type: enum, required: true, allowed: [planned, done], default: planned }
      - { name: endAcknowledgedAt, type: instant, required: false, constraints: "set when the owner answers the Time's up sheet (Done or Not yet)" }
      - { name: createdAt, type: instant, required: true }
      - { name: updatedAt, type: instant, required: true }
      - { name: version, type: integer, required: true, min: 1, default: 1 }
    constraints:
      - "endsAt > startsAt, endsAt - startsAt >= 15 minutes (BR5.3, BR5.6)."
      - "localDate equals the owner's today at creation (BR5.2)."
    relationships:
      - { to: Task, cardinality: "many to 1", direction: "TimeBlock schedules one Task" }
      - { to: FocusSession, cardinality: "1 to many", direction: "TimeBlock hosts Pomodoro sessions" }

  - name: FocusSession
    owned_by: Focus
    description: One countdown. A Pomodoro (from a block) or a 2-minute countdown (from an unsorted task). The 5-minute start (justStart) arrives in U3.
    attributes:
      - { name: sessionId, type: identifier, required: true, unique: true }
      - { name: ownerId, type: reference, required: true, references: Owner.ownerId }
      - { name: taskId, type: reference, required: true, references: Task.taskId }
      - { name: blockId, type: reference, required: false, references: TimeBlock.blockId, constraints: "required for pomodoro, empty for twoMinute" }
      - { name: mode, type: enum, required: true, allowed: [pomodoro, twoMinute, justStart] }
      - { name: startedAt, type: instant, required: true, constraints: "server clock; the only basis for remaining time" }
      - { name: pausedAt, type: instant, required: false, constraints: "used from U3" }
      - { name: pausedMs, type: integer, required: true, min: 0, default: 0, constraints: "total paused time; used from U3" }
      - { name: endedAt, type: instant, required: false }
      - { name: endReason, type: enum, required: false, allowed: [stopped, blockEnded, countdownFinished, replaced] }
      - { name: answer, type: enum, required: false, allowed: [done, notYet], constraints: "twoMinute only" }
      - { name: phase, type: enum, required: true, allowed: [focus, break, longBreak, ended], constraints: "derived from elapsed time (BR6.3); stored copy is a cache" }
      - { name: completedIntervals, type: integer, required: true, min: 0, constraints: "derived from elapsed time; stored copy is a cache" }
    constraints:
      - "At most one FocusSession per owner has an empty endedAt (BR6.6)."
    relationships:
      - { to: Task, cardinality: "many to 1", direction: "FocusSession is about one Task" }
      - { to: TimeBlock, cardinality: "many to 0..1", direction: "Pomodoro runs inside one TimeBlock" }

  - name: DeviceSubscription
    owned_by: Notifications
    description: One browser or home-screen app that can receive system notifications.
    attributes:
      - { name: subscriptionId, type: identifier, required: true, unique: true }
      - { name: ownerId, type: reference, required: true, references: Owner.ownerId }
      - { name: endpoint, type: url, required: true, unique: true, constraints: "https only; from the browser's push service" }
      - { name: publicKeys, type: key-pair-public-parts, required: true }
      - { name: createdAt, type: instant, required: true }
      - { name: lastFailureAt, type: instant, required: false }
    relationships:
      - { to: Owner, cardinality: "many to 1", direction: "DeviceSubscription belongs to Owner" }

  - name: ScheduledAlert
    owned_by: Notifications
    description: One notification due at a time. Identified for replacement by sourceKey.
    attributes:
      - { name: alertId, type: identifier, required: true, unique: true }
      - { name: ownerId, type: reference, required: true, references: Owner.ownerId }
      - { name: sourceKey, type: text, required: true, constraints: "unique per owner among pending alerts, e.g. 'block:<blockId>:end', 'focus:<sessionId>:phase'" }
      - { name: kind, type: enum, required: true, allowed: [intervalEnd, blockEnd, messageWindow] }
      - { name: fireAt, type: instant, required: true }
      - { name: title, type: text, required: true, max: 120 }
      - { name: url, type: text, required: true, allowed: ["/today", "/focus"] }
      - { name: status, type: enum, required: true, allowed: [pending, sent, cancelled, failed], default: pending }
      - { name: attempts, type: integer, required: true, min: 0, default: 0 }
      - { name: sentAt, type: instant, required: false }
    relationships:
      - { to: Owner, cardinality: "many to 1", direction: "ScheduledAlert is for one Owner" }

  - name: OutboxEvent
    owned_by: "app shell (shared by emitting components)"
    description: An internal event written in the same transaction as the change that caused it, then delivered in-process (C2, ADR-010).
    attributes:
      - { name: eventId, type: identifier, required: true, unique: true }
      - { name: type, type: enum, required: true, allowed: [TaskCategoryChanged, TaskCompleted, BlockCreated, BlockChanged, BlockRemoved, PreferencesChanged] }
      - { name: aggregateId, type: identifier, required: true }
      - { name: payload, type: structured, required: true, constraints: "ids and times only, never task text" }
      - { name: occurredAt, type: instant, required: true }
      - { name: deliveredAt, type: instant, required: false }
      - { name: attempts, type: integer, required: true, min: 0, default: 0 }
      - { name: lastError, type: text, required: false, constraints: "no task text" }
    relationships: []
```

## Summary

| Entity | Owner component | Purpose in this slice |
|---|---|---|
| Owner | Account | The one account; time zone defines "today" |
| Session | Account | Signed-in browser, cookie plus CSRF token |
| SignInAttempt | Account | Slows down repeated failed sign-ins |
| Task | Tasks | Captured item; unsorted, doNow or completed here |
| CaptureBatch | Tasks | Makes a repeated capture submit harmless |
| TimeBlock | Planning | A doNow task's slot today |
| FocusSession | Focus | Pomodoro or 2-minute countdown, timed from its start instant |
| DeviceSubscription | Notifications | A device that can receive notifications |
| ScheduledAlert | Notifications | A due notification, replaceable by source key |
| OutboxEvent | app shell | Internal events, saved with the change that caused them |

Additions to the component catalogue, all additive: `Session.csrfToken`,
`Session.lastSeenAt`, `Session.revokedAt`, `SignInAttempt`, `Task.captureBatchKey`,
`Task.capturePosition`, `CaptureBatch.contentDigest`, `TimeBlock.endAcknowledgedAt`,
`FocusSession.endReason`, `FocusSession.answer`, `ScheduledAlert.url`,
`ScheduledAlert.attempts`, `ScheduledAlert.sentAt`, and `OutboxEvent` (the C2
outbox table).
