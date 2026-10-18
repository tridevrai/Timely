# Story-to-Unit Map: Timely

Every story in `user-stories/stories.md` is assigned to exactly one implementing
unit. "Order in unit" is the suggested build order inside that unit (story
dependencies from `stories.md`); the order of units is decided in Delivery Planning.

| Story | Title | Unit ID | Directory | Order in unit | Notes |
|---|---|---|---|---|---|
| US1.1 | Sign in to my own space | U1 | u1-core-loop-slice | 1 |  |
| US1.2 | Keep my work when my session expires | U2 | u2-sort-and-lists | 1 | browser drafts used on every screen |
| US2.1 | Brain dump many items at once | U1 | u1-core-loop-slice | 2 |  |
| US2.2 | Never lose or duplicate a capture | U1 | u1-core-loop-slice | 3 |  |
| US3.1 | Do a 2-minute task right away | U1 | u1-core-loop-slice | 4 |  |
| US3.2 | Cancel a 2-minute countdown early | U2 | u2-sort-and-lists | 2 |  |
| US4.1 | Sort an item into Do Now | U1 | u1-core-loop-slice | 5 |  |
| US4.2 | Delegate an item | U2 | u2-sort-and-lists | 3 |  |
| US4.3 | Defer an item to a future day | U2 | u2-sort-and-lists | 4 |  |
| US4.4 | Drop an item, with a way back | U2 | u2-sort-and-lists | 5 |  |
| US4.5 | Change my mind about a category | U2 | u2-sort-and-lists | 6 | also touches Planning blocks (built in U1) |
| US4.6 | Write the first small step | U2 | u2-sort-and-lists | 7 |  |
| US4.7 | Act on a delegate follow-up | U2 | u2-sort-and-lists | 8 |  |
| US5.1 | Time-block a task today | U1 | u1-core-loop-slice | 6 |  |
| US5.2 | See overlapping blocks | U3 | u3-day-and-focus | 1 |  |
| US5.3 | Carry unscheduled Do Now tasks to tomorrow | U3 | u3-day-and-focus | 2 |  |
| US6.1 | Focus on a block with a Pomodoro | U1 | u1-core-loop-slice | 7 |  |
| US6.2 | Keep the timer right when the device sleeps | U3 | u3-day-and-focus | 3 |  |
| US6.3 | Pause, resume, or stop a session | U3 | u3-day-and-focus | 4 |  |
| US6.4 | Just start for 5 minutes | U3 | u3-day-and-focus | 5 |  |
| US7.1 | Get told when an interval or block ends | U1 | u1-core-loop-slice | 8 |  |
| US7.2 | Decide what happens when a block ends | U3 | u3-day-and-focus | 6 |  |
| US8.1 | See my Google Calendar events on today's plan | U4 | u4-calendar-sync | 1 |  |
| US8.2 | Put my time blocks in Google Calendar | U4 | u4-calendar-sync | 2 | uses Planning block events from U1 |
| US8.3 | Pick up changes made in Google Calendar | U4 | u4-calendar-sync | 3 |  |
| US8.4 | Keep planning when Google is unreachable | U4 | u4-calendar-sync | 4 |  |
| US9.1 | Get reminded to check messages at set times | U5 | u5-reminders-and-insights | 1 |  |
| US9.2 | See a short verse when I start planning | U5 | u5-reminders-and-insights | 2 |  |
| US10.1 | See how my week went | U5 | u5-reminders-and-insights | 3 | reads done blocks from U3 |
| US10.2 | Set a limit for my open list | U5 | u5-reminders-and-insights | 4 |  |
| US11.1 | Clear, predictable lists | U2 | u2-sort-and-lists | 9 |  |
| US11.2 | Same data on Mac and iPhone | U3 | u3-day-and-focus | 7 | rule applies to Task and TimeBlock; reused by later units |

## Cross-cutting stories

| Story | Units involved | How |
|---|---|---|
| US1.2 | U2 (owner), every unit's screens | the browser draft mechanism is built in U2 and used by later screens |
| US4.5 | U2 (owner), U1 | U2 emits TaskCategoryChanged; Planning (from U1) removes the block |
| US11.2 | U3 (owner), U2, U4 | version-conflict prompt built in U3; Task and TimeBlock updates in other units use it |
| US8.2 | U4 (owner), U1 | relies on Planning's block events created in U1 |
| US10.1 | U5 (owner), U3 | counts blocks marked done by U3's end-of-block decisions |

## Coverage check

- Stories enumerated: 32. Assigned: 32. Unassigned: 0.
- U1: 8 stories, U2: 9 stories, U3: 7 stories, U4: 4 stories, U5: 4 stories. Every unit has at least one story.
