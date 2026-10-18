# User Stories Questions

Story plan (proposed): stories are written in the "As a ..., I want ..., so
that ..." form, each small enough to build and test in 1–3 days, with 3–6
Given/When/Then acceptance criteria and a Must / Should / Could / Won't
priority. The questions below settle who the stories are written for, how they
are grouped, and two behaviors the requirements review left open.

## Question 1: Who should the stories be written for?

A. One persona: you, the owner, who plans and does the work (suggested)
B. One persona with two modes: you while planning (calm, deciding) and you while focusing (in a block, wanting no distractions)
C. You, plus a "delegate" persona for people you hand tasks to, kept for the later accounts phase
X. Other (please specify)

[Answer]: A

## Question 2: How should the stories be grouped?

A. By step of the daily loop: capture, 2-minute rule, sort, plan today, focus, wrap-up, plus supporting groups (sign-in, calendar, reminders, summary) (suggested)
B. By requirement area, one group per FR1–FR13
C. By release: thin first slice, then the rest
X. Other (please specify)

[Answer]: A

## Question 3: Which stories belong in the thin first slice?

Context: you chose to build a thin end-to-end slice first and approve it before
the rest. Everything in the slice is a Must Have.

A. Sign-in, brain dump, 2-minute rule, sort, time-block today, Pomodoro with notification at the end. Google Calendar sync comes right after the slice (suggested)
B. The same as A, plus Google Calendar sync inside the slice
C. Only brain dump, sort, time-block and Pomodoro, with no sign-in yet (run it locally)
X. Other (please specify)

[Answer]: A

## Question 4: How should the other features be prioritized?

A. Must: Google Calendar sync, delegate and defer details, next-step prompt and 5-minute start, end-of-block wrap-up. Should: communication reminders, weekly summary. Could: daily verse (suggested)
B. Must: everything in the requirements; priorities only set the build order
C. Same as A, but the daily verse is a Should
X. Other (please specify)

[Answer]: C

## Question 5: What can you do with a delegate follow-up reminder?

Context: on a delegated task's follow-up date, a reminder appears on that day's
plan. The requirements don't yet say what you can do with it.

A. Three actions: "Done" (they finished it, so it's complete), "Push date" (pick a new follow-up date), "Take it back" (it moves to Do Now) (suggested)
B. Only "Done" and "Push date"
X. Other (please specify)

[Answer]: A

## Question 6: If a time block is changed in Timely and its event is changed in Google Calendar before they sync, which change wins?

A. The most recent change wins, and Timely tells me it replaced the other one (suggested)
B. Timely always wins and overwrites Google
C. Google always wins and overwrites Timely
D. Neither: Timely shows both versions and asks me to choose
X. Other (please specify)

[Answer]: A

## Question 7: Can a 2-minute countdown be cancelled before it ends?

A. Yes: cancelling returns the item to the unsorted list unchanged (suggested)
B. Yes, and cancelling also asks if it's done
C. No, it always runs to the end
X. Other (please specify)

[Answer]: X - A and B combined
