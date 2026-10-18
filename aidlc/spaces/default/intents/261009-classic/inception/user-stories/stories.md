# User Stories

All stories are written for **P1 The Owner** (`personas.md`). Each is sized for
1–3 days of work and has Given/When/Then acceptance criteria. Priorities follow
`user-stories-questions.md` Q3–Q4. Stories marked **[Slice]** make up the thin
end-to-end slice, which is built and approved before everything else (team.md
Walking Skeleton).

Story origins outside `requirements.md`: US4.7 (follow-up reminder actions) comes
from Q5, US8.3's conflict rule from Q6, and US3.2 (cancelling the countdown) from Q7.
They close gaps the requirements review raised.

## Thin first slice

US1.1 → US2.1 → US2.2 → US3.1 → US4.1 → US5.1 → US6.1 → US7.1

Together they run the core loop once from start to finish: sign in, dump, 2-minute
check, sort into Do Now, block it today, focus in a Pomodoro, and get notified when it ends.

---

## Group 1: Sign-in

### US1.1 Sign in to my own space [Slice]
As the owner, I want to sign in before seeing anything, so that my tasks and plans stay private on a server reachable from anywhere.
- **Priority**: Must Have · **Traces**: FR1.1, FR1.2 · **Depends on**: none
- AC1.1.1 Given I am signed out, When I open any page, Then I see the sign-in screen and no task data.
- AC1.1.2 Given correct credentials, When I sign in, Then I land on today's plan.
- AC1.1.3 Given wrong credentials, When I sign in, Then I see "Sign-in failed" without being told which part was wrong, and no data is shown.
- INVEST: independent foundation story; testable at the HTTP and screen level.

### US1.2 Keep my work when my session expires
As the owner, I want unsaved input kept when my session expires, so that I never lose a half-written brain dump.
- **Priority**: Must Have · **Traces**: FR1.3 · **Depends on**: US1.1
- AC1.2.1 Given my session expired while typing, When I sign back in, Then the text I typed is still there.
- AC1.2.2 Given my session expired, When I submit a change, Then it is not lost: it is applied after I sign in again, or I am told it was not saved.
- AC1.2.3 Given I sign out on purpose, When I sign back in, Then no stale draft from before is shown.

## Group 2: Capture

### US2.1 Brain dump many items at once [Slice]
As the owner, I want to type or paste everything on my mind, one item per line, so that I can empty my head in one go without deciding anything yet.
- **Priority**: Must Have · **Traces**: FR2.1, FR2.2 · **Depends on**: US1.1
- AC2.1.1 Given the capture screen, When I paste 12 lines including 2 blank ones, Then 10 unsorted items are created.
- AC2.1.2 Given a line of only spaces, When I submit, Then no item is created for it.
- AC2.1.3 Given an item containing emoji or `<script>` text, When it is shown anywhere, Then it appears as plain text and is never executed.
- AC2.1.4 Given a single 2,000-character line, When I submit, Then it is saved whole and shown without breaking the layout.

### US2.2 Never lose or duplicate a capture [Slice]
As the owner, I want captured items saved immediately and only once, so that I can trust the list completely.
- **Priority**: Must Have · **Traces**: FR2.3, FR2.4 · **Depends on**: US2.1
- AC2.2.1 Given I captured 3 items, When I close the tab and reopen the app, Then all 3 are in my unsorted list.
- AC2.2.2 Given I double-tap submit, When the request is sent twice, Then each item exists once.
- AC2.2.3 Given the server is unreachable, When I submit, Then I am told the items were not saved and my typed text stays in the box.

## Group 3: 2-minute rule

### US3.1 Do a 2-minute task right away [Slice]
As the owner, I want to start a quick item with a 2-minute countdown, so that tiny tasks get done instead of clogging the list.
- **Priority**: Must Have · **Traces**: FR3.1, FR3.2, FR3.3, FR3.4 · **Depends on**: US2.1
- AC3.1.1 Given an unsorted item, When I start it as 2-minute, Then a 2:00 countdown runs and no sort category is required.
- AC3.1.2 Given the countdown reaches 0:00, When I answer "Done", Then the item is complete and leaves the unsorted list.
- AC3.1.3 Given the countdown reaches 0:00, When I answer "Not yet", Then the item is back in the unsorted list unchanged.

### US3.2 Cancel a 2-minute countdown early
As the owner, I want to stop a 2-minute countdown before it ends, so that I'm not stuck waiting when I finish early or get interrupted.
- **Priority**: Must Have · **Traces**: FR3.3 (refined by Q7) · **Depends on**: US3.1
- AC3.2.1 Given a running countdown, When I cancel it, Then the countdown stops and I am asked whether the item is done.
- AC3.2.2 Given I cancelled and answer "Done", Then the item is marked complete.
- AC3.2.3 Given I cancelled and answer "Not yet", Then the item returns to the unsorted list unchanged.

## Group 4: Sort

### US4.1 Sort an item into Do Now [Slice]
As the owner, I want to mark an item "Do Now", so that it becomes something I can schedule today.
- **Priority**: Must Have · **Traces**: FR4.1, FR4.2 · **Depends on**: US2.1
- AC4.1.1 Given an unsorted item, When I choose Do Now, Then it leaves the unsorted list and appears in Do Now.
- AC4.1.2 Given an item already sorted, When I look at the unsorted list, Then it is not there; each item has exactly one category.
- AC4.1.3 Given an empty unsorted list, When I open it, Then I see an empty state pointing me to the brain dump.

### US4.2 Delegate an item
As the owner, I want to record who I handed a task to and when to check back, so that delegated work doesn't fall through the cracks.
- **Priority**: Must Have · **Traces**: FR4.1, FR4.3 · **Depends on**: US4.1
- AC4.2.1 Given an unsorted item, When I choose Delegate with a name and a follow-up date, Then it appears in the Delegated list ordered by follow-up date.
- AC4.2.2 Given I leave the name or date empty, When I save, Then I am told which field is missing and the item stays unsorted with my input kept.
- AC4.2.3 Given a follow-up date in the past, When I save, Then I am asked for today or a later date.

### US4.3 Defer an item to a future day
As the owner, I want to set a later date on an item, so that it disappears until it actually matters.
- **Priority**: Must Have · **Traces**: FR4.4 · **Depends on**: US4.1
- AC4.3.1 Given an item deferred to 2026-11-03, When my local date becomes 2026-11-03, Then it is back in the unsorted list.
- AC4.3.2 Given a defer date of today or earlier, When I save, Then I am told to choose a future date.
- AC4.3.3 Given a deferred item, When I open the Deferred list, Then it shows with its date, ordered by date.

### US4.4 Drop an item, with a way back
As the owner, I want to drop items I won't do and still be able to restore them, so that dropping feels safe.
- **Priority**: Must Have · **Traces**: FR4.5 · **Depends on**: US4.1
- AC4.4.1 Given an unsorted item, When I choose Drop, Then it moves to the Archive and leaves every active list.
- AC4.4.2 Given an archived item, When I restore it, Then it is back in the unsorted list with its text unchanged.
- AC4.4.3 Given an empty Archive, When I open it, Then I see an empty state saying nothing has been dropped.

### US4.5 Change my mind about a category
As the owner, I want to move an item to a different category later, so that my sort can follow reality.
- **Priority**: Must Have · **Traces**: FR4.6 · **Depends on**: US4.2, US4.3, US4.4
- AC4.5.1 Given a Do Now item, When I change it to Defer with a future date, Then it leaves Do Now and appears in Deferred.
- AC4.5.2 Given a Delegated item, When I change it to Do Now, Then its delegate name and follow-up date are cleared.
- AC4.5.3 Given an item in a time block today, When I change it away from Do Now, Then I am told the block will be removed, and the block is removed only if I confirm.

### US4.6 Write the first small step
As the owner, I want to be prompted for the smallest physical next step of a Do Now task, so that broad tasks become easy to start.
- **Priority**: Must Have · **Traces**: FR5.1, FR5.2 · **Depends on**: US4.1
- AC4.6.1 Given I sort a task with no next step into Do Now, When the prompt appears, Then I can enter a next step or skip.
- AC4.6.2 Given a task has a next step, When it appears in Do Now or on today's plan, Then the next step is shown with it.
- AC4.6.3 Given I skipped the prompt, When I open the task later, Then I can still add a next step.

### US4.7 Act on a delegate follow-up
As the owner, I want to close, postpone, or take back a delegated task when its follow-up comes up, so that every delegation ends in a decision.
- **Priority**: Must Have · **Traces**: FR4.3 (refined by Q5) · **Depends on**: US4.2
- AC4.7.1 Given a delegated task whose follow-up date is today, When I open today's plan, Then a follow-up reminder for it is shown.
- AC4.7.2 Given a follow-up reminder, When I choose "Done", Then the task is complete and the reminder disappears.
- AC4.7.3 Given a follow-up reminder, When I choose "Push date" and pick a later date, Then the reminder disappears today and returns on the new date.
- AC4.7.4 Given a follow-up reminder, When I choose "Take it back", Then the task moves to Do Now and the delegate name is cleared.

## Group 5: Plan today

### US5.1 Time-block a task today [Slice]
As the owner, I want to place a Do Now task into a time slot on today's timeline, so that it has a real place in my day.
- **Priority**: Must Have · **Traces**: FR6.1, FR6.2 · **Depends on**: US4.1
- AC5.1.1 Given a Do Now task, When I place it 09:00–10:30 today, Then it shows on today's timeline at that time in my time zone.
- AC5.1.2 Given a block that ends before it starts, When I save, Then I see an error and nothing is saved.
- AC5.1.3 Given a block that would run past midnight, When I save, Then I am told blocks must end today.
- AC5.1.4 Given I try to plan a different day, When I look for it, Then only today can be planned.

### US5.2 See overlapping blocks
As the owner, I want overlapping blocks flagged, so that I notice when my day is overbooked.
- **Priority**: Must Have · **Traces**: FR6.4 · **Depends on**: US5.1
- AC5.2.1 Given blocks 09:00–10:00 and 09:30–10:30, When I view today, Then both are saved and both are flagged as overlapping with a text label, not only a color.
- AC5.2.2 Given blocks 09:00–10:00 and 10:00–11:00, When I view today, Then neither is flagged.

### US5.3 Carry unscheduled Do Now tasks to tomorrow
As the owner, I want Do Now tasks I didn't schedule to still be there tomorrow, so that nothing silently disappears.
- **Priority**: Must Have · **Traces**: FR6.5 · **Depends on**: US5.1
- AC5.3.1 Given an unblocked Do Now task at midnight, When I open the app the next day, Then it is still in Do Now.
- AC5.3.2 Given a daylight-saving change overnight, When the new day starts, Then the carry-over happens once, at local midnight.

## Group 6: Focus

### US6.1 Focus on a block with a Pomodoro [Slice]
As the owner, I want to start a Pomodoro from a time block, so that I work in focused intervals on the task I planned.
- **Priority**: Must Have · **Traces**: FR8.1, FR8.2 · **Depends on**: US5.1
- AC6.1.1 Given a time block, When I start a Pomodoro, Then a 25-minute focus interval starts, linked to that block's task.
- AC6.1.2 Given a focus interval ends, When the break starts, Then it lasts 5 minutes.
- AC6.1.3 Given I completed 4 focus intervals, When the 4th ends, Then the next break is 15 minutes.

### US6.2 Keep the timer right when the device sleeps
As the owner, I want the timer to stay accurate when my phone locks or the tab is in the background, so that I can trust it.
- **Priority**: Must Have · **Traces**: FR8.3 · **Depends on**: US6.1
- AC6.2.1 Given a running interval, When I lock my phone for 10 minutes and come back, Then about 10 minutes less is remaining.
- AC6.2.2 Given a running interval, When I reload the page, Then the timer resumes with the right time remaining.
- AC6.2.3 Given an interval ended while the device slept, When I come back, Then the app shows that it ended and moves to the next step.

### US6.3 Pause, resume, or stop a session
As the owner, I want to pause, resume, or stop a Pomodoro, so that real interruptions don't break the timer.
- **Priority**: Must Have · **Traces**: FR8.4 · **Depends on**: US6.1
- AC6.3.1 Given a running interval, When I pause for 3 minutes and resume, Then the remaining time is what it was when I paused.
- AC6.3.2 Given a running session, When I stop it, Then no further intervals or notifications happen for it.
- AC6.3.3 Given I tap pause twice quickly, When it settles, Then the session is paused once, not paused then resumed.

### US6.4 Just start for 5 minutes
As the owner, I want a 5-minute "just start" timer on a Do Now task, so that I can get past the urge to avoid it.
- **Priority**: Must Have · **Traces**: FR5.3 · **Depends on**: US4.1, US6.1
- AC6.4.1 Given a Do Now task, When I choose "Just start", Then a 5-minute timer runs for it.
- AC6.4.2 Given the 5 minutes end, When I choose "Keep going", Then a Pomodoro session starts for that task.
- AC6.4.3 Given the 5 minutes end, When I choose "Stop", Then the task stays in Do Now.

## Group 7: Wrap-up

### US7.1 Get told when an interval or block ends [Slice]
As the owner, I want a sound and a notification when an interval or block ends, even if the app is in the background, so that I can keep my head down without watching the clock.
- **Priority**: Must Have · **Traces**: FR9.1, FR9.3 · **Depends on**: US6.1
- AC7.1.1 Given the app is in the background and notifications are allowed, When an interval ends, Then I get a notification with sound within 30 seconds of the end time.
- AC7.1.2 Given notifications are not allowed on this device, When I open the app, Then it says notifications are off and how to turn them on.
- AC7.1.3 Given notifications are not allowed, When a block ended while I was away, Then the end-of-block prompt is shown the next time I open the app.

### US7.2 Decide what happens when a block ends
As the owner, I want to say whether a task is done, needs more time, or should move when its block ends, so that my plan stays honest.
- **Priority**: Must Have · **Traces**: FR9.2 · **Depends on**: US7.1
- AC7.2.1 Given a block ended, When I choose "Done", Then the task is complete and the block is marked done.
- AC7.2.2 Given a block ended, When I choose "Needs more time", Then I can extend the block or add a new block later today.
- AC7.2.3 Given a block ended, When I choose "Reschedule", Then I can send the task back to Do Now or defer it to a future date.

## Group 8: Calendar

### US8.1 See my Google Calendar events on today's plan
As the owner, I want my existing Google Calendar events shown on today's timeline, so that I plan around meetings.
- **Priority**: Must Have · **Traces**: FR6.3, FR7.1 · **Depends on**: US5.1
- AC8.1.1 Given I connect my Google account and choose a calendar, When I open today, Then today's events from it are shown read-only.
- AC8.1.2 Given I have not connected Google, When I open today, Then the plan works with my own blocks and offers to connect.
- AC8.1.3 Given an all-day event, When I open today, Then it is shown at the top, not as a 24-hour block.

### US8.2 Put my time blocks in Google Calendar
As the owner, I want each Timely block written to my Google calendar and kept up to date, so that my calendar shows when I'm busy.
- **Priority**: Must Have · **Traces**: FR7.2 · **Depends on**: US8.1
- AC8.2.1 Given I create a 09:00–10:00 block, When sync completes, Then exactly one matching event exists in Google Calendar.
- AC8.2.2 Given I move or resize a block, When sync completes, Then the Google event has the new time.
- AC8.2.3 Given I delete a block, When sync completes, Then its Google event is removed.

### US8.3 Pick up changes made in Google Calendar
As the owner, I want edits made in Google Calendar to Timely's events to show up in Timely, with a clear rule when both sides changed, so that the two never silently disagree.
- **Priority**: Must Have · **Traces**: FR7.3 (conflict rule from Q6) · **Depends on**: US8.2
- AC8.3.1 Given I move a Timely event in Google Calendar, When Timely next syncs, Then the block moves to match.
- AC8.3.2 Given I delete a Timely event in Google Calendar, When Timely next syncs, Then the block is removed and the task goes back to Do Now.
- AC8.3.3 Given the block was changed in Timely and its event was changed in Google since the last sync, When they sync, Then the more recent change is kept and Timely tells me which change it replaced.

### US8.4 Keep planning when Google is unreachable
As the owner, I want Timely to keep working when Google Calendar is down or my connection is revoked, so that a sync problem never blocks my day.
- **Priority**: Must Have · **Traces**: FR7.4, FR7.5 · **Depends on**: US8.2
- AC8.4.1 Given Google is unreachable, When I create a block, Then it is saved in Timely and labelled "not yet synced", with the last sync time shown.
- AC8.4.2 Given connectivity returns, When the retry runs, Then the block appears in Google exactly once.
- AC8.4.3 Given my Google access was revoked, When Timely tries to sync, Then I am told and offered to reconnect, and no blocks are lost.

## Group 9: Reminders

### US9.1 Get reminded to check messages at set times
As the owner, I want reminders at my communication windows, so that I check messages in batches instead of all day.
- **Priority**: Should Have · **Traces**: FR10.1, FR10.2 · **Depends on**: US7.1
- AC9.1.1 Given no windows are set, When I open the settings, Then the windows default to 10:00, 13:00 and 16:00.
- AC9.1.2 Given a window at 13:00, When it is 13:00, Then I get a reminder to check messages.
- AC9.1.3 Given windows are set, When I view today's plan, Then they do not appear as blocks or calendar events.

### US9.2 See a short verse when I start planning
As the owner, I want a short verse or reminder the first time I open today's plan each day, so that I start the day with intention.
- **Priority**: Should Have · **Traces**: FR11.1, FR11.2 · **Depends on**: US5.1
- AC9.2.1 Given it's my first visit to today's plan today, When it opens, Then one verse from the built-in set is shown.
- AC9.2.2 Given I dismiss it, When I open today's plan again today, Then it is not shown.
- AC9.2.3 Given a new day, When I first open today's plan, Then a verse is shown again.

## Group 10: Summary

### US10.1 See how my week went
As the owner, I want a simple weekly summary, so that I can tell whether Timely is actually helping.
- **Priority**: Should Have · **Traces**: FR12.1, FR12.3 · **Depends on**: US7.2
- AC10.1.1 Given 5 days planned and 18 of 24 blocks done this week, When I open the summary, Then it shows 5 planned days and 18/24 blocks done.
- AC10.1.2 Given no blocks this week, When I open the summary, Then it shows "no blocks planned", not 0% or an error.
- AC10.1.3 Given the week runs Monday to Sunday in my time zone, When a block is done at 23:30 Sunday, Then it counts in that week.

### US10.2 Set a limit for my open list
As the owner, I want to set the most open items I'm comfortable with and see how I'm doing against it, so that the list never grows back into a massive one.
- **Priority**: Should Have · **Traces**: FR12.1, FR12.2 · **Depends on**: US10.1
- AC10.2.1 Given I set the limit to 20, When I have 23 open items, Then the summary shows 23 of 20 and that I'm over.
- AC10.2.2 Given I enter 0, a negative number, or text, When I save, Then I am told to enter a whole number of 1 or more and the old limit is kept.

## Group 11: Lists across devices

### US11.1 Clear, predictable lists
As the owner, I want every list to have a fixed order and a helpful empty state, so that I always know where things are and what to do next.
- **Priority**: Must Have · **Traces**: FR13.1, FR13.2 · **Depends on**: US4.2, US4.3, US4.4
- AC11.1.1 Given the unsorted list, When I view it, Then items are in capture order, ties broken by creation time.
- AC11.1.2 Given the Delegated and Deferred lists, When I view them, Then they are ordered by follow-up or defer date, ties broken by creation time.
- AC11.1.3 Given any empty list, When I open it, Then it says it's empty and names the next action.

### US11.2 Same data on Mac and iPhone
As the owner, I want my Mac and iPhone to show the same data and never silently overwrite each other, so that I can switch devices freely.
- **Priority**: Must Have · **Traces**: FR13.3 · **Depends on**: US1.1
- AC11.2.1 Given I sort an item on my phone, When I refresh on my Mac, Then the Mac shows the new category.
- AC11.2.2 Given I edit an item on my Mac after my phone already changed it, When I save on the Mac, Then I am told it changed elsewhere and my edit is not saved over the newer one without my choice.
