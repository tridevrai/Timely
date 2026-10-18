# Units Generation Questions

This step groups the 9 building blocks into **units of work**: chunks that are
built, tested, and approved one at a time during construction. You chose to
build a thin end-to-end slice first, so the first unit has to be that slice.
Build order beyond that is decided in the next planning step, not here.

**Proposed units** (Questions 2 and 3 confirm or change them):

| Unit | What it delivers | Stories |
|---|---|---|
| U1 Core loop slice | Sign in → capture → 2-minute → sort into Do Now → block it today → Pomodoro → end notification, with just the screens and server parts those steps need | US1.1, US2.1, US2.2, US3.1, US4.1, US5.1, US6.1, US7.1 |
| U2 Sort and lists | Delegate, Defer, Drop/restore, changing category, next step, follow-ups, cancelling the 2-minute countdown, list order and empty states, keeping drafts when a session expires | US1.2, US3.2, US4.2–US4.7, US11.1 |
| U3 Day and focus | Overlaps, carry-over at midnight, timer accuracy through sleep and reload, pause/resume, the 5-minute start, end-of-block decisions, Mac and iPhone conflict handling | US5.2, US5.3, US6.2–US6.4, US7.2, US11.2 |
| U4 Calendar sync | Connecting Google, showing events, writing blocks, picking up changes and conflicts, working offline | US8.1–US8.4 |
| U5 Reminders and insights | Message-check reminders, the daily verse, the weekly summary, the open-list limit | US9.1, US9.2, US10.1, US10.2 |

U2, U3, U4 build on U1. U5 builds on U3, because the summary counts end-of-block decisions.

## Question 1: How should Timely be deployed?

A. As one app: a single server that also serves the web pages. The building blocks stay separate inside it and talk through defined interfaces, never each other's data (suggested; fits one owner on one self-hosted server)
B. Two parts: the web pages hosted separately from the server
C. Separate services per building block
X. Other (please specify)

[Answer]: A

## Question 2: How should the work be divided into units?

A. By feature slice, as in the table: each unit cuts through the screens and the server for a set of stories, starting with the core loop (suggested)
B. By building block: one unit per block (Account, Tasks, Planning, ...), with the slice assembled across them
X. Other (please specify)

[Answer]: A

## Question 3: How many units?

A. Five, as in the table (suggested)
B. Two: the core loop slice, then everything else
C. More and smaller: split U2 and U3 into two units each
X. Other (please specify)

[Answer]: A
