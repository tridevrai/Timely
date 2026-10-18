# Functional Design Questions: U1 Core loop slice

This step turns the walking skeleton (sign in → brain dump → 2-minute rule →
Do Now → time-block today → Pomodoro → end notification) into exact rules and
step-by-step behaviour. Most of it is already settled by the requirements,
stories, components (ADR-001 to ADR-010) and contracts (C1 to C5). These are
the few gaps those documents leave open for this slice.

Smaller corners are settled with defaults you can correct after reading the
design (for example: capture lines are trimmed of surrounding spaces; only one
owner exists; a 2-minute countdown left unanswered asks "Done?" again the next
time the app opens).

## Question 1: How should the single owner account be created the first time?

Context: version 1 has exactly one owner (FR1.2) and no sign-up screen. The
password must never be stored in the code or the repository.

A. A one-time command run on the server (for example `timely create-owner --email you@example.com`) that asks for the password and refuses if an owner already exists (suggested)
B. A setup screen shown on first visit that works only while no owner exists, then disappears for good
C. Email and password hash read from environment variables when the server starts
X. Other (please specify)

[Answer]: A

## Question 2: In this slice, what should the "Time's up" sheet offer when a time block ends?

Context: the full end-of-block choices (Done, Needs more time, Reschedule) are
built in Bolt 3 (US7.2). The skeleton still needs a way to finish the loop.

A. "Done" or "Not yet": Done completes the task and marks the block done; Not yet leaves the task in Do Now and the block as it was (suggested)
B. Only "Done" and a dismiss button; dismissing changes nothing
X. Other (please specify)

[Answer]: A

## Question 3: What should happen to a running Pomodoro when its time block's end time arrives?

Context: a Pomodoro started from a block (US6.1) can still be mid-interval when
the block ends. Without a rule you would get two overlapping alerts.

A. The Pomodoro stops at the block's end; you get one alert, and the "Time's up" sheet opens (suggested)
B. The Pomodoro keeps cycling until you stop it; the block-end alert fires separately
C. The Pomodoro finishes its current focus interval, then stops
X. Other (please specify)

[Answer]: A

## Question 4: Can you place a block that starts earlier today than the current time?

Context: only today can be planned (FR6.1), but part of today has already passed.

A. Yes, as long as it ends after the current time (so you can block "now until 10:30") (suggested)
B. Yes, anywhere today, even entirely in the past (useful for logging what you did)
C. No, a block must start at or after the current time
X. Other (please specify)

[Answer]: A

## Question 5: What are the smallest block and the time grid for typed or dragged times?

Context: the Mac layout drags in 15-minute steps (`refined-mockups/interaction-spec.md`).
Typed times on the iPhone need the same rule so both devices agree.

A. Times on 5-minute marks, blocks at least 15 minutes long; dragging still snaps to 15 minutes (suggested)
B. Times on 15-minute marks only, blocks at least 15 minutes long
C. Any minute, blocks at least 5 minutes long
X. Other (please specify)

[Answer]: A

## Question 6: What happens if you start a countdown while another one is already running?

Context: the 2-minute countdown and the Pomodoro both live in Focus (ADR-002).
Two running at once would mean two sets of alerts.

A. Only one countdown at a time: starting another asks whether to stop the current one first (suggested)
B. Starting another silently stops the current one
C. Allow several at once
X. Other (please specify)

[Answer]: A
