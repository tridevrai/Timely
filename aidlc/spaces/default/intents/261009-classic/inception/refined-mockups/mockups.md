# Refined Mockups: Timely

Mid-fidelity mockups for every screen in version 1, designed from
`inception/user-stories/stories.md` and `inception/requirements-analysis/requirements.md`.
There are no rough mockups (the classic plan skips Ideation), so nothing here is
carried from an earlier sketch.

**Design decisions** (from `refined-mockups-questions.md`):
- Phone: bottom tab bar with four labelled tabs: Today, Capture, Sort, Lists. Minimalist visual style: one primary action per screen, generous white space, no decoration. [Q1, FU1]
- Sorting: one item at a time on a card, five big choices, advances automatically. [Q2]
- Time-blocking: drag and resize on Mac; tap a task, then pick times on iPhone; both keyboard-operable. [Q3]
- Pomodoro: full-screen focus view. [Q4]
- Mac: two panels, lists left and today's timeline right. [Q5]
- Light/dark follows the device; no appearance setting. [Q6]
- Mac shortcuts: N capture, S sort, T today, Space pause/resume, 1–5 sort choices. [Q7]

Every screen below shows its empty, loading, error and populated states where
they differ. Story IDs in brackets show what each screen covers.

---

## Screen map

```
Sign in ──► Today ◄──────────────┐
             │  ▲                 │
   ┌─────────┼──┼──────┬──────────┤
   ▼         ▼  │      ▼          ▼
 Capture    Sort      Lists     Focus (full screen, from a block)
             │        ├─ Delegated
             ▼        ├─ Deferred
        2-minute      ├─ Archive
        countdown     ├─ Weekly summary
                      └─ Settings (time zone, message windows,
                                   open-list limit, Google)
```

Text fallback: Sign in leads to Today. From Today the tab bar reaches Capture,
Sort and Lists; Focus opens full-screen from a time block. Sort can open the
2-minute countdown. Lists contains Delegated, Deferred, Archive, Weekly summary
and Settings.

---

## M1 Sign in [US1.1, US1.2]

```
┌──────────────────────────────┐
│                              │
│            Timely            │
│                              │
│  Email                       │
│  [__________________________]│
│  Password                    │
│  [__________________________]│
│                              │
│  [        Sign in         ]  │
│                              │
└──────────────────────────────┘
```
- **Error**: "Sign-in failed. Check your email and password." under the button; fields keep what was typed except the password.
- **Session expired** (over any screen): a sheet "You've been signed out. Sign in to continue. Your unsaved text is kept." After sign-in, the user returns to the same screen with the draft restored.

## M2 Today (phone) [US5.1, US5.2, US5.3, US4.7, US8.1, US8.4, US9.2]

```
┌──────────────────────────────┐
│ Today · Mon 19 Oct      ⟳ 2m │  ← last Google sync
├──────────────────────────────┤
│ ┃ "Redeem the time."   Eph 5 │  ← daily verse, first open only
│ ┃                    [Dismiss]│
├──────────────────────────────┤
│ Follow-ups (1)               │
│  Ask Sam re: invoice         │
│  [Done] [Push date] [Take back]│
├──────────────────────────────┤
│ 08:00                        │
│ 09:00 ┌ Tax: download June ─┐│
│       │ statement   ▶ Focus ││
│ 10:00 └─────────────────────┘│
│ 11:00 ░ Standup (Google) ░░░ │  ← read-only event
│ 12:00 ┌ Write proposal ─────┐│
│       │ ⚠ Overlaps          ││
│ 13:00 └─────────────────────┘│
│                              │
│ Do Now · not scheduled (3) ▸ │
├──────────────────────────────┤
│  Today  Capture  Sort  Lists │
│   ●                      (4) │  ← Sort badge = unsorted count
└──────────────────────────────┘
```
- Timeline starts at the current hour and scrolls; the current time is a thin line labelled "now".
- **Empty**: "Nothing planned yet. Pick a Do Now task to give it a time." with the Do Now list expanded.
- **Not yet synced**: a block shows "Not yet synced" text under its title; the header reads "Google: last synced 09:12, retrying".
- **Google disconnected**: banner "Google Calendar access stopped. [Reconnect]". Blocks stay.
- **Loading**: timeline skeleton (grey bars) for up to 5 seconds, then "Still loading…" text.

## M3 Place a task in a block (phone) [US5.1, US5.2]

```
┌──────────────────────────────┐
│ ← Schedule                   │
│ Tax: download June statement │
│                              │
│ Start   [ 09:00 ▾ ]          │
│ End     [ 10:30 ▾ ]          │
│                              │
│ ⚠ Overlaps "Write proposal"  │
│   12:00–13:00 — it's allowed │
│                              │
│ [        Save block       ]  │
└──────────────────────────────┘
```
- End before start: "End time must be after the start time." The Save button stays enabled; the error appears on save, next to the End field.
- End past midnight: "Blocks must end today."
- Opening from an empty slot pre-fills Start with that slot.

## M4 Capture [US2.1, US2.2]

```
┌──────────────────────────────┐
│ Capture                      │
│ Everything on your mind,     │
│ one per line.                │
│ ┌──────────────────────────┐ │
│ │call dentist              │ │
│ │fix bike light            │ │
│ │taxes                     │ │
│ │                          │ │
│ └──────────────────────────┘ │
│ 3 items                      │
│ [       Add to inbox      ]  │
├──────────────────────────────┤
│  Today  Capture  Sort  Lists │
└──────────────────────────────┘
```
- The counter ignores blank lines.
- **Success**: the box clears and "3 items added · Sort now" appears inline for 5 seconds.
- **Error**: "Couldn't save. Your text is still here. Try again." The text is kept.
- The button is disabled while saving to stop double submits.

## M5 Sort (one item at a time) [US3.1, US4.1–US4.6]

```
┌──────────────────────────────┐
│ Sort                 3 of 10 │
│ ┌──────────────────────────┐ │
│ │                          │ │
│ │  Work on taxes           │ │
│ │                          │ │
│ └──────────────────────────┘ │
│ [1  2-minute — do it now   ] │
│ [2  Do Now                 ] │
│ [3  Delegate               ] │
│ [4  Defer                  ] │
│ [5  Drop                   ] │
│                     [Skip ›] │
├──────────────────────────────┤
│  Today  Capture  Sort  Lists │
└──────────────────────────────┘
```
- **Do Now** → "What's the smallest next step?" field with [Save] and [Skip]. [US4.6]
- **Delegate** → Name and Follow-up date fields; missing fields are named inline. [US4.2]
- **Defer** → date picker with past and today disabled. [US4.3]
- **Drop** → moves immediately; an "Undo" toast shows for 5 seconds. [US4.4]
- After each choice the next card slides in (no motion if reduced motion is on).
- **Empty**: "Inbox clear. Nothing to sort." [Go to Today]

## M6 2-minute countdown [US3.1, US3.2]

```
┌──────────────────────────────┐
│ call dentist                 │
│                              │
│            1:42              │
│                              │
│ [ Cancel ]                   │
└──────────────────────────────┘
          at 0:00 or on Cancel
┌──────────────────────────────┐
│ Is it done?                  │
│ [ Done ]      [ Not yet ]    │
└──────────────────────────────┘
```

## M7 Focus (full screen) [US6.1–US6.4, US7.1, US7.2]

```
┌──────────────────────────────┐
│ Focus · 2 of 4               │
│                              │
│ Tax: download June statement │
│ Next: log into the portal    │
│                              │
│           18:24              │
│                              │
│   [ Pause ]     [ Stop ]     │
└──────────────────────────────┘
```
- **Break**: same layout, heading "Break · 5 min" (or "Long break · 15 min" after the 4th interval), task text dimmed.
- **Just start (5 min)**: heading "Just start · 5 min"; at the end: [Keep going] starts a Pomodoro, [Stop] returns.
- **Block ended** sheet: "Time's up for Tax: download June statement" with [Done] [Needs more time] [Reschedule]. Needs more time offers "+15 min", "+30 min" or "New block today"; Reschedule offers "Back to Do Now" or "Defer to…".
- **Notifications off**: a one-line note under the timer: "Notifications are off; Timely can only alert you while open. [How to turn on]".

## M8 Lists [US4.2–US4.5, US4.7, US11.1]

```
┌──────────────────────────────┐
│ Lists                        │
│ Do Now               4     ▸ │
│ Delegated            2     ▸ │
│ Deferred             5     ▸ │
│ Archive             11     ▸ │
│ Weekly summary             ▸ │
│ Settings                   ▸ │
├──────────────────────────────┤
│  Today  Capture  Sort  Lists │
└──────────────────────────────┘
```
List detail (example: Delegated):
```
┌──────────────────────────────┐
│ ← Delegated                  │
│ Ask Sam re: invoice          │
│   Sam · follow up Tue 20 Oct │
│ Book venue                   │
│   Priya · follow up Fri 23 Oct│
└──────────────────────────────┘
```
- Tapping an item opens its detail with "Change category" and its fields.
- Changing a scheduled Do Now task away asks: "This removes its 09:00 block today. Continue?" [US4.5]
- **Empty** examples: Delegated "Nothing delegated." · Deferred "Nothing waiting for a later day." · Archive "Nothing dropped." Each has a [Go to Sort] link.

## M9 Weekly summary [US10.1, US10.2]

```
┌──────────────────────────────┐
│ ← This week · 19–25 Oct      │
│ Days planned        5 of 7   │
│ Blocks done         18 / 24  │
│ Open list           23 / 20  │
│                 Over by 3    │
└──────────────────────────────┘
```
- No blocks: "Blocks done: no blocks planned."
- Every number has a text label; "Over by 3" is text, not only red.

## M10 Settings [US8.1, US9.1, US10.2]

```
┌──────────────────────────────┐
│ ← Settings                   │
│ Time zone   [Pacific/Auckland▾]│
│ Message check times          │
│   [10:00] [13:00] [16:00] [+]│
│ Open-list limit   [ 20 ]     │
│ Google Calendar              │
│   Connected · Work calendar  │
│   [Change] [Disconnect]      │
│ [Sign out]                   │
└──────────────────────────────┘
```
- Limit validation: "Enter a whole number of 1 or more." The old value is kept.

## M11 Mac two-panel layout [all of the above, Q5]

```
┌─────────────────────┬────────────────────────────────────┐
│ Timely   N  S  T    │ Today · Mon 19 Oct     Google ⟳ 2m │
│ ─────────────────── │ ────────────────────────────────── │
│ Unsorted (4)  [Sort]│ 09:00 ┌ Tax: download June ─────┐  │
│  call dentist       │       │ statement      ▶ Focus  │  │
│  fix bike light     │ 10:00 └─────────────────────────┘  │
│ Do Now (3)          │ 11:00 ░ Standup (Google) ░░░░░░░░  │
│  ⠿ Write proposal   │ 12:00                              │
│  ⠿ Email landlord   │ 13:00 ┄┄ drop here ┄┄┄┄┄┄┄┄┄┄┄┄┄   │
│ Lists ▸             │                                    │
└─────────────────────┴────────────────────────────────────┘
```
- Drag a Do Now task (⠿ handle) onto the timeline; drag the block's bottom edge to resize. Keyboard alternative: focus a task, press Enter → "Schedule" with time fields (same as M3).
- Capture opens as a panel above the left list (N); Sort opens as the card view in the left panel (S).
- Focus still takes the whole window.
- At widths under 768 px the phone layout is used.
