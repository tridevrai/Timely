# Accessibility Checklist: Timely (WCAG 2.1 AA)

Target set by requirements NFR6: everything works by keyboard, text contrast
meets WCAG 2.1 AA, status is never shown by colour alone. Each line names
where the design meets it and how Build and Test checks it.

## Perceivable
- [ ] Text contrast ≥ 4.5:1 and UI components ≥ 3:1 in light and dark modes (design-system-mapping Colour). Check: automated contrast audit of both modes.
- [ ] Status never by colour alone: overlap (⚠ + "Overlaps"), not yet synced (text), Google events (hatch + "(Google)"), over the open-list limit ("Over by n"), errors (text). Check: greyscale review of M2, M9.
- [ ] Icons in the tab bar always have visible labels. Check: M2/M4/M5/M8.
- [ ] Layout works at 200% zoom and with the OS large text setting; no horizontal scroll at 320 px width. Check: manual zoom test.

## Operable
- [ ] Every action is reachable by keyboard, including drag alternatives: Enter on a task opens "Schedule"; Alt+Up/Down moves a block (interaction-spec Timeline). Check: full keyboard-only run of flow F1.
- [ ] Visible focus ring (2 px, `accent`, ≥ 3:1) on every interactive element. Check: tab through every screen.
- [ ] Logical focus order; after sorting, focus moves to the new card; after closing a sheet, focus returns to its trigger.
- [ ] Single-letter shortcuts (N, S, T, 1–5) are ignored while typing in a field (WCAG 2.1.4).
- [ ] No keyboard traps; Focus view exits via Stop; dialogs close with Esc.
- [ ] Touch targets ≥ 44×44 px; primary buttons 48 px.
- [ ] Timing: Pomodoro and countdowns can be paused or stopped; toasts pause while hovered or focused; no session timeout drops typed text (US1.2).
- [ ] Motion: reduced-motion setting turns off card and sheet transitions.

## Understandable
- [ ] `lang="en"` on the page.
- [ ] Every input has a visible label; errors name the field and the fix ("End time must be after the start time.").
- [ ] Destructive actions: Drop has Undo; removing a scheduled block asks for confirmation (US4.4, US4.5).
- [ ] Navigation is the same on every screen (tab bar on phone, panel header on Mac).

## Robust
- [ ] Native elements first: `button`, `textarea`, `input type="time"` and `type="date"`, `nav`.
- [ ] Live regions: `aria-live="polite"` for capture results, sort advance, timer phase changes, toasts; the countdown is not announced every second.
- [ ] Screen reader pass with VoiceOver on macOS Safari and iOS Safari through flow F1.

## Out of scope for version 1
Captions and transcripts (the app has no audio or video beyond a notification sound, which is always paired with a visible notification).
