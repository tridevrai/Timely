# Design System Mapping: Timely

There is no existing design system or component library; the technology stack
is not chosen yet (team.md Code Style). This file defines the minimal tokens
and component set version 1 needs, in keeping with the minimalist style the
owner asked for [Q1/FU1] and the "no productivity theater" principle [requirements Q14].
When the stack is chosen, these map onto its native or chosen components.

## Tokens

### Colour (follows device light/dark automatically [Q6])
| Token | Light | Dark | Use |
|---|---|---|---|
| `bg` | #FFFFFF | #111214 | page background |
| `surface` | #F4F5F7 | #1C1E21 | cards, panels |
| `text` | #16181B | #ECEDEF | body text (≥ 15:1 on bg) |
| `text-muted` | #5A616B | #A3A9B2 | secondary text (≥ 4.5:1 on bg and surface) |
| `accent` | #2457D6 | #7FA4FF | primary buttons, focus ring, current tab |
| `warning` | #8A5A00 | #F0B44C | overlap and "not yet synced" labels (always with icon and text) |
| `danger` | #B3261E | #FF8A80 | errors (always with text) |
| `event` | #E6E8EC | #2A2D31 | Google events (plus hatch pattern) |

All text and UI pairs to be checked at 4.5:1 (text) and 3:1 (UI) in both modes
before the build is accepted; values above are starting points.

### Type
System font stack (`-apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif`), so
the app looks native on Mac and iPhone with nothing to download.
| Token | Size / line height | Use |
|---|---|---|
| `title` | 22/28 semibold | screen titles |
| `body` | 17/24 | lists, cards |
| `small` | 14/20 | metadata, timeline hours |
| `timer` | 64–120 tabular numerals | countdowns |
Text scales with the browser and OS text size; layouts tested at 200% zoom.

### Spacing and shape
Spacing scale 4, 8, 16, 24, 32, 48 px. Corner radius 10 px for cards and
buttons. Touch targets at least 44×44 px; primary buttons 48 px tall.

### Motion
Card advance and sheet open ≤ 200 ms ease-out; none when `prefers-reduced-motion` is set.

## Component inventory

| Component | Screens | Stories | Notes |
|---|---|---|---|
| TabBar | M2, M4, M5, M8 | navigation for all | phone only |
| CaptureBox | M4, M11 | US2.1, US2.2 | |
| SortCard | M5, M11 | US3.1, US4.1–US4.6 | |
| Timeline | M2, M11 | US5.1, US5.2, US8.1, US8.4 | |
| BlockSheet (schedule/edit) | M3 | US5.1, US4.5 | time pickers use native `input type="time"` |
| FocusTimer | M6, M7 | US3.1, US3.2, US6.1–US6.4 | |
| EndOfBlockSheet | M7 | US7.2 | |
| FollowUpRow | M2 | US4.7 | |
| VerseCard | M2 | US9.2 | dismissible |
| ListRow / ListDetail | M8 | US4.2–US4.5, US11.1 | |
| SummaryStat | M9 | US10.1, US10.2 | |
| SettingsForm | M10 | US8.1, US9.1, US10.2 | |
| Toast | all | US4.4, US8.3 | |
| Banner | M2 | US8.4, US7.1 | Google disconnected, notifications off |
| ConfirmDialog | M8 | US4.5 | only for removing a scheduled block |
| EmptyState | M2, M5, M8, M9 | US11.1, US4.1 | text plus one action |

## Not included in version 1
Illustrations, custom icon sets beyond a small system-style set, themes,
colour pickers, animation beyond the two transitions above [requirements Q14].
