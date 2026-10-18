# Domain Design Questions

This step splits Timely into its building blocks: the separate pieces of code,
what each one is responsible for, and which piece owns which data. It does not
choose the technology or how the pieces are deployed; that comes later.

**Proposed building blocks** (the questions below confirm or change them):

| Block | What it's responsible for |
|---|---|
| Account | Your sign-in, sessions, and settings (time zone, message windows, open-list limit) |
| Tasks | Every task from capture to done: unsorted, the 2-minute rule, Do Now, Delegate, Defer, Drop, the next step, follow-ups |
| Planning | Today's time blocks: placing, moving, overlaps, carrying unscheduled work over |
| Focus | All countdowns: Pomodoro, the 2-minute countdown, the 5-minute start |
| Calendar Sync | The connection to Google Calendar: reading events, writing blocks, resolving conflicts |
| Notifications | Every alert: interval and block ends, message-check reminders |
| Insights | The weekly summary |
| Web App | The screens from the mockups |

## Question 1: Should all task categories live in one block?

A. Yes: one Tasks block owns a task through its whole life, with the category (Do Now, Delegate, Defer, Drop) as part of the task. Moving a task between categories is a single change (suggested)
B. Separate blocks for the inbox, delegated tasks, and deferred tasks, with tasks handed between them
X. Other (please specify)

[Answer]: A

## Question 2: Where should the 2-minute countdown live?

A. In Focus, together with the Pomodoro and the 5-minute start; Focus tells Tasks when an item is done (suggested)
B. In Tasks, since it's part of sorting
X. Other (please specify)

[Answer]: A

## Question 3: Should time blocks be separate from tasks?

A. Yes: Planning owns time blocks, and each block points to one task. A task can exist without a block (suggested)
B. No: a task stores its own start and end time, and there's no separate block
X. Other (please specify)

[Answer]: A

## Question 4: How should Google Calendar be connected?

A. A separate Calendar Sync block that translates between Google's events and Timely's blocks. It owns the Google connection and the link between each block and its Google event, so nothing else in Timely needs to know how Google works (suggested)
B. Planning talks to Google directly
X. Other (please specify)

[Answer]: A

## Question 5: Who sends the notifications?

A. One Notifications block. Focus, Planning, Account, and Tasks ask it to schedule or cancel alerts, and it owns the device subscriptions and the delivery logic (suggested)
B. Each block sends its own notifications
X. Other (please specify)

[Answer]: A

## Question 6: How should the weekly summary be produced?

A. Insights works it out when you open it, by reading from Tasks and Planning. Nothing extra is stored (suggested)
B. Insights keeps its own running totals, updated each time a block or task changes
X. Other (please specify)

[Answer]: A

## Question 7: Where should the settings (time zone, message windows, open-list limit) live?

A. In Account, as your preferences (suggested)
B. In a separate Settings block
C. Each setting with the block that uses it (message windows in Notifications, the limit in Insights)
X. Other (please specify)

[Answer]: A

## Question 8: Where should the daily verses come from?

A. A fixed list built into the Web App; it remembers whether you've dismissed today's verse (suggested)
B. Stored on the server, so the list can be changed without a new release
X. Other (please specify)

[Answer]: B

## Question 9: Where should the rules live (for example "defer dates must be in the future" or "the newest change wins")?

A. On the server. The Web App shows the screens and checks fields as you type, but the server makes the final decision, so the Mac and iPhone always agree. If the server can't be reached, the app shows it's offline (suggested)
B. Mostly in the Web App, working fully offline and syncing to the server later
X. Other (please specify)

[Answer]: A

## Follow-up 1: How will the verse list be changed once it's on the server?

Context: Question 8 chose storing the verses on the server so the list can
change without a new release. The approved requirements assumed a fixed list
with no editing screen in version 1 (to avoid extra settings).

A. No editing screen in version 1: the list is seeded on the server and I change it directly in the data when I want to (suggested)
B. Add a simple screen in Settings to add, edit, and remove verses
X. Other (please specify)

[Answer]: A
