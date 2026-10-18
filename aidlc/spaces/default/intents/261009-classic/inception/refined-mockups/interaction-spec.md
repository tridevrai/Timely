# Interaction Specification: Timely

Companion to `mockups.md`. Flows first, then component specs in the
`component-spec-template.md` format. Breakpoints: phone < 768 px, tablet
768–1023 px (phone layout with wider content), desktop ≥ 1024 px (two-panel layout).

## User flows

### F1 Core loop (the thin first slice)
Persona: P1 Owner · Trigger: start of the day
1. Sign in (M1) → credentials → Today (M2).
2. Capture tab (M4) → types lines → "Add to inbox" → items saved, counter shown.
3. Sort tab (M5) → per card: 2-minute (M6) or Do Now (+ next-step prompt).
4. Today (M2/M3, or drag on M11) → place Do Now task 09:00–10:30.
5. Block "▶ Focus" → Focus view (M7) → 25/5 intervals.
6. Interval/block end → notification + sound → "Time's up" sheet → Done.
Success: task marked done, block shows "Done".
Error paths: save failure in step 2 keeps text and offers retry; session expiry at any step restores the draft after sign-in; notifications off shows the end sheet next time the app opens.

### F2 Delegate follow-up
Trigger: follow-up date is today → Today shows "Follow-ups (n)" → Done / Push date (date picker, past disabled) / Take back (moves to Do Now, clears name).

### F3 Calendar conflict
Trigger: same block changed in Timely and Google before sync → the newer change wins → a toast on Today: "Updated from Google Calendar: 'Write proposal' moved to 14:00 (your 12:00 change was replaced)" with [View].

### F4 Drop with undo
Drop on a card → item archived → toast "Dropped 'fix bike light' · Undo" for 5 s → Undo restores it to the same position in the sort queue.

## Keyboard shortcuts (Mac) [Q7]

| Key | Action | Active when |
|---|---|---|
| N | Open Capture | not typing in a field |
| S | Start sorting | not typing in a field |
| T | Go to Today | not typing in a field |
| 1–5 | Sort choice on the current card | Sort card focused |
| Space | Pause / resume | Focus view |
| Esc | Close sheet or dialog | sheet open |
| Cmd+Enter | Submit capture | Capture text area |

Single-letter shortcuts are ignored while a text field has focus (WCAG 2.1.4).

---

## SortCard

| Field | Value |
|---|---|
| Component | SortCard |
| Description | Shows one unsorted item and the five sort choices |
| Category | input |

### States
| State | Description | Trigger |
|---|---|---|
| default | Item text and five choice buttons | an unsorted item exists |
| detail | Inline fields for the chosen category (next step, delegate, defer date) | Do Now / Delegate / Defer chosen |
| saving | Buttons disabled, "Saving…" | choice submitted |
| error | "Couldn't save. Try again." The item stays on screen | save failed |
| empty | "Inbox clear. Nothing to sort." | no unsorted items |

### Props / Inputs
| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| item | object | yes | — | id, text, createdAt |
| position | number | yes | — | index for "3 of 10" |
| total | number | yes | — | unsorted count |
| onSort | function | yes | — | (category, details) |
| onSkip | function | no | — | moves item to the end of the queue |

### Responsive Behaviour
| Breakpoint | Behaviour |
|---|---|
| mobile (<768px) | full width; buttons stacked, 48 px tall |
| tablet (768–1024px) | same, max width 560 px centred |
| desktop (>1024px) | inside the left panel |

### Accessibility
| Requirement | Implementation |
|---|---|
| ARIA role | `region` labelled "Sort item 3 of 10"; native buttons |
| Keyboard interaction | Tab through choices; keys 1–5; Enter to confirm detail fields |
| Label / aria-label | Visible button text, e.g. "Do Now" |
| Contrast ratio | WCAG AA (4.5:1 text, 3:1 UI components) |
| Screen reader | On advance, announces the new item text (`aria-live="polite"`) |
| Focus management | After a choice, focus moves to the new card's heading |

### Usage Example
```
<SortCard item={item} position={3} total={10} onSort={sort} onSkip={skip} />
```

---

## Timeline

| Field | Value |
|---|---|
| Component | Timeline |
| Description | Today's hours with Timely blocks, Google events and the now line |
| Category | display / input |

### States
| State | Description | Trigger |
|---|---|---|
| default | Blocks and events positioned by time | data loaded |
| loading | Grey skeleton bars; "Still loading…" after 5 s | fetching |
| empty | "Nothing planned yet" plus the Do Now list | no blocks or events |
| error | "Couldn't load today. [Retry]" | load failed |
| stale | Header shows last sync time and "retrying" | Google unreachable |
| dragging | Drop target line shown at the 15-minute slot | desktop drag |

### Props / Inputs
| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| date | string | yes | today | ISO date in the owner's time zone |
| blocks | array | yes | [] | Timely blocks (id, start, end, task, syncState, done) |
| events | array | no | [] | read-only Google events |
| onCreateBlock | function | yes | — | (taskId, start, end) |
| onUpdateBlock | function | yes | — | move/resize |

### Responsive Behaviour
| Breakpoint | Behaviour |
|---|---|
| mobile (<768px) | full width; tap a block to open it; no drag |
| tablet (768–1024px) | same as mobile |
| desktop (>1024px) | right panel; drag to create/move, bottom-edge resize, 15-minute snapping |

### Accessibility
| Requirement | Implementation |
|---|---|
| ARIA role | `list` of blocks in time order; each block a `listitem` with a button |
| Keyboard interaction | Arrow keys move between blocks; Enter opens a block; Alt+Up/Down moves a focused block by 15 min |
| Label / aria-label | "09:00 to 10:30, Tax: download June statement, overlaps Write proposal" |
| Contrast ratio | WCAG AA; Google events use a hatch pattern plus the "(Google)" text, not only a colour |
| Screen reader | Sync state and overlap read as text |
| Focus management | After creating a block, focus goes to the new block |

### Usage Example
```
<Timeline date="2026-10-19" blocks={blocks} events={events} onCreateBlock={create} onUpdateBlock={update} />
```

---

## FocusTimer

| Field | Value |
|---|---|
| Component | FocusTimer |
| Description | Full-screen Pomodoro, break, 2-minute and 5-minute countdowns |
| Category | feedback |

### States
| State | Description | Trigger |
|---|---|---|
| running | Large countdown, Pause and Stop | session started or resumed |
| paused | "Paused" label, Resume and Stop | Pause or Space |
| break | "Break · 5 min" or "Long break · 15 min" | focus interval ended |
| ended | "Time's up" sheet with decisions | block or countdown ended |
| caught-up | "This interval ended while you were away" + next step | device woke after the end |

### Props / Inputs
| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| mode | "pomodoro" \| "two-minute" \| "just-start" | yes | — | which countdown |
| startedAt | string | yes | — | server-recorded start instant |
| pausedMs | number | no | 0 | total paused time |
| task | object | yes | — | text and next step |

Remaining time is always computed from `startedAt` and `pausedMs`, never by counting ticks (FR8.3).

### Responsive Behaviour
| Breakpoint | Behaviour |
|---|---|
| mobile (<768px) | full screen; countdown 64 px |
| tablet (768–1024px) | full screen; countdown 96 px |
| desktop (>1024px) | full window; countdown 120 px |

### Accessibility
| Requirement | Implementation |
|---|---|
| ARIA role | `timer` for the countdown, which is not announced every second |
| Keyboard interaction | Space pause/resume; Esc does nothing (avoids an accidental exit); Stop needs Enter |
| Label / aria-label | "18 minutes 24 seconds left in focus interval 2 of 4" |
| Contrast ratio | WCAG AA |
| Screen reader | Announces only phase changes and the 1-minute mark (`aria-live="polite"`) |
| Focus management | On open, focus on Pause; on end, focus on the first decision button |

### Usage Example
```
<FocusTimer mode="pomodoro" startedAt={s} pausedMs={0} task={task} />
```

---

## CaptureBox

| Field | Value |
|---|---|
| Component | CaptureBox |
| Description | Multi-line text area that turns each non-blank line into an item |
| Category | input |

### States
| State | Description | Trigger |
|---|---|---|
| default | Empty area with hint "Everything on your mind, one per line." | open |
| typing | Live count of non-blank lines | input |
| saving | Button disabled, "Saving…" | submit |
| success | Clears; "3 items added · Sort now" for 5 s | saved |
| error | "Couldn't save. Your text is still here." | save failed |

### Props / Inputs
| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| draft | string | no | "" | restored after session expiry |
| onSubmit | function | yes | — | (lines, idempotencyKey) |

### Responsive Behaviour
| Breakpoint | Behaviour |
|---|---|
| mobile (<768px) | full-height text area above the keyboard |
| tablet (768–1024px) | same |
| desktop (>1024px) | panel above the left list; Cmd+Enter submits |

### Accessibility
| Requirement | Implementation |
|---|---|
| ARIA role | native `textarea` with a `label` |
| Keyboard interaction | Enter adds a new line; Cmd+Enter or the button submits |
| Label / aria-label | Visible label "Capture" |
| Contrast ratio | WCAG AA |
| Screen reader | Item count and the success or error message via `aria-live="polite"` |
| Focus management | Focus stays in the box after success |

### Usage Example
```
<CaptureBox draft={draft} onSubmit={capture} />
```

---

## Toast

| Field | Value |
|---|---|
| Component | Toast |
| Description | Short status with an optional action (Undo, View) |
| Category | feedback |

### States
| State | Description | Trigger |
|---|---|---|
| default | Message and optional action, 5 s | event |
| paused | Timer stops while hovered or focused | hover/focus |

### Props / Inputs
| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| message | string | yes | — | plain text |
| action | object | no | — | label + handler |
| durationMs | number | no | 5000 | auto-dismiss time |

### Responsive Behaviour
| Breakpoint | Behaviour |
|---|---|
| mobile (<768px) | above the tab bar, full width |
| tablet (768–1024px) | same |
| desktop (>1024px) | bottom-left of the left panel |

### Accessibility
| Requirement | Implementation |
|---|---|
| ARIA role | `status` |
| Keyboard interaction | The action is reachable by Tab; the toast does not dismiss while it has focus |
| Label / aria-label | message text |
| Contrast ratio | WCAG AA |
| Screen reader | Announced politely |
| Focus management | Focus is not moved to the toast |

### Usage Example
```
<Toast message="Dropped 'fix bike light'" action={{label:"Undo", onClick:undo}} />
```

---

## TabBar

| Field | Value |
|---|---|
| Component | TabBar |
| Description | Phone navigation: Today, Capture, Sort (with count), Lists |
| Category | navigation |

### States
| State | Description | Trigger |
|---|---|---|
| default | Four labelled icons; the current one is filled and labelled | any screen |
| badge | Sort shows its unsorted count when above 0 | unsorted items exist |
| hidden | Not shown | Focus view |

### Props / Inputs
| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| current | string | yes | "today" | active tab |
| unsortedCount | number | yes | 0 | badge value |

### Responsive Behaviour
| Breakpoint | Behaviour |
|---|---|
| mobile (<768px) | fixed bottom, 56 px plus the safe area |
| tablet (768–1024px) | same |
| desktop (>1024px) | replaced by the left-panel header links |

### Accessibility
| Requirement | Implementation |
|---|---|
| ARIA role | `nav` with links; `aria-current="page"` on the active tab |
| Keyboard interaction | Tab and Enter |
| Label / aria-label | "Sort, 4 unsorted" |
| Contrast ratio | WCAG AA |
| Screen reader | Reads the label and count |
| Focus management | Navigating moves focus to the new screen's heading |

### Usage Example
```
<TabBar current="today" unsortedCount={4} />
```
