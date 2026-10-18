# Requirements Analysis Questions

Source: `initial_product_idea.md` (the "Overwhelm to Execution" handover
document) and the practices you approved. The core loop is: brain dump →
2-minute rule and Do Now / Delegate / Defer / Drop sort → time-block on the
calendar → Pomodoro focus session. These questions settle what the app itself
must do. Pick a letter, or write your own answer under X.

## Question 1: Where should the app run?

Context: the handover document is Apple-centric (Mac and iPhone). This choice
shapes everything built later.

A. A web app that works in the browser on Mac and iPhone (can be added to the iPhone home screen)
B. A native Mac app only, iPhone later
C. Native apps for both Mac and iPhone
D. A web app first, native apps later if it proves useful
X. Other (please specify)

[Answer]: D

## Question 2: Who will use it?

A. Only me, on my own devices (no accounts or sign-in needed beyond protecting my data)
B. Me now, but it should support other people signing up later
C. Me and a few people I share tasks with (for example, people I delegate to)
X. Other (please specify)

[Answer]: C

## Question 3: Where should your tasks and plans be stored?

A. Only on one device, no syncing
B. Synced between my Mac and iPhone so both always show the same list and day plan
C. Stored on a server I host, reachable from any device
X. Other (please specify)

[Answer]: C

## Question 4: How should time-blocking relate to your existing calendar?

Context: the document compares TickTick and Apple Reminders + Calendar. Time
blocks could live only in this app, or appear in the calendar you already use.

A. Time blocks live only inside this app
B. The app shows my existing calendar events (read-only) so I can plan around them, and keeps its own blocks
C. The app reads my calendar and also writes time blocks into it (two-way)
D. Start with A, add calendar integration later
X. Other (please specify)

[Answer]: C

## Question 5: How far ahead should you be able to time-block?

Context: the document leans on focusing on the current 24-hour window
(Matthew 6:34).

A. Today only; everything else waits in the deferred list
B. Today and tomorrow
C. Any day, with a week view
X. Other (please specify)

[Answer]: A

## Question 6: What should each sort category do?

Context: after the 2-minute rule, each remaining item goes to Do Now,
Delegate, Defer/Schedule, or Drop.

A. Do Now: ready to be time-blocked today. Delegate: record who it went to and a follow-up date. Defer: pick a future date, and it reappears that day. Drop: archived (recoverable), not deleted
B. Same as A, but Drop deletes permanently
C. Keep it simple: the categories are just labels, with no dates or follow-ups
X. Other (please specify)

[Answer]: A

## Question 7: How should 2-minute tasks be handled?

A. Mark it "2-minute", do it right away, then tick it off; it never enters the sort
B. Show a 2-minute countdown when I start one, then ask if it's done
C. Treat it as an ordinary Do Now task
X. Other (please specify)

[Answer]: B

## Question 8: Should the app help break big tasks into a first small step?

Context: the document's fix for procrastination is micro-stepping (the smallest
physical next step) plus the 5-minute commitment.

A. Yes: when a task looks broad, prompt me for its next physical step, and offer a 5-minute "just start" timer
B. Yes, let me add a next step to a task, but don't prompt me
C. Not in the first version
X. Other (please specify)

[Answer]: A

## Question 9: How should the Pomodoro focus session work?

A. Fixed 25 minutes on, 5 minutes off, linked to the time block I'm working in
B. Same as A, plus a longer break after every 4 sessions
C. Adjustable lengths, defaulting to 25/5
X. Other (please specify)

[Answer]: X - go with B for now and later phase C

## Question 10: What should happen when a time block or Pomodoro ends?

A. Notify me (sound and notification) even if the app is in the background, then ask whether the task is done, needs more time, or should be rescheduled
B. Show it only when I have the app open
X. Other (please specify)

[Answer]: A

## Question 11: Should the app help with communication batching?

Context: the document recommends opening email and chat only 3 times a day
(for example 10 AM, 1 PM, 4 PM).

A. Yes: let me set communication windows and show them as blocks in my day plan
B. Yes, as reminders only
C. Not in the first version
X. Other (please specify)

[Answer]: B

## Question 12: Should the faith-based foundations appear in the app?

Context: the document cites Ephesians 5:15–16, Proverbs 6:6–8, and Matthew 6:34.

A. Yes, show a short verse or reminder at the start of each day's planning
B. No, keep the app neutral
X. Other (please specify)

[Answer]: A

## Question 13: How will you know the app is working for you?

A. I plan my day in it at least 5 days a week, and most planned blocks get finished
B. My open list stays below a number I choose
C. Both A and B, shown as a simple weekly summary
D. No tracking; I'll judge by feel
X. Other (please specify)

[Answer]: C

## Question 14: What must the first version leave out?

Context: the document warns about "productivity theater": spending time on
settings and polish instead of the work. Choose everything that should wait.

A. Themes, colors, and appearance settings
B. Recurring tasks
C. Tags, projects, and folders
D. Statistics beyond the basics
X. Other (please specify, for example "all of the above")

[Answer]: X - all of the above

## Follow-up 1: What can the people you share tasks with do?

Context: Question 2 chose "me and a few people I share tasks with", and
Question 6 records who a delegated task went to. This decides whether those
people need their own accounts in the app.

A. They get their own accounts: a task I delegate appears in their list, they can mark it done, and I see the status
B. They don't use the app; delegating just records their name and a follow-up date for me
C. Start with B, add A later
X. Other (please specify)

[Answer]: C

## Follow-up 2: Which calendar should the app read and write?

Context: Question 4 chose two-way calendar integration. The app has to connect
to a specific calendar service.

A. Apple iCloud Calendar
B. Google Calendar
C. Both
X. Other (please specify)

[Answer]: B
