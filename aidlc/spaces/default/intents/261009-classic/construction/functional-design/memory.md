<!-- INVARIANT: examples are single-line HTML comments so a fresh template parses to total=0 (MEMORY_EMPTY). Do NOT un-comment or split across lines. t100 guards this. -->
> This file is kept up to date automatically while the stage runs. Add observations at the review step, not by editing here directly.

## Interpretations
<!-- example: 2026-05-29T10:14:32Z — chose REST over GraphQL; the consuming team only needs CRUD, revisit if subscriptions land -->
- 2026-10-18T22:08:27Z [unit u1-core-loop-slice] — Treated the "ended, unanswered" block as a derived view, not a stored status; TimeBlock.status stays planned|done as in the C1 contract, and a new endAcknowledgedAt attribute decides whether the Time's up sheet is shown again.
- 2026-10-18T22:08:27Z [unit u1-core-loop-slice] — Derived Pomodoro phase and interval count from elapsed time (startedAt, pausedMs) rather than storing them as truth; the stored copies are caches, which keeps FR8.3 (sleep, reload) correct by construction.

## Deviations
<!-- example: 2026-05-29T10:14:32Z — skipped the optional caching layer the stage prose suggested; the dataset is small enough that it adds risk -->
- 2026-10-18T22:08:27Z [unit u1-core-loop-slice] — Did not produce frontend-components.md: the unit is tagged service, so the screen behaviour lives in functional-spec.md workflows and the refined mockups' interaction spec instead.
- 2026-10-18T22:08:27Z [unit u1-core-loop-slice] — Added attributes and one entity beyond the component catalogue (SignInAttempt, OutboxEvent, Session.csrfToken, FocusSession.endReason/answer, TimeBlock.endAcknowledgedAt); all additive and listed at the end of entities.md.

## Tradeoffs
<!-- example: 2026-05-29T10:14:32Z — picked TDD over BDD this run; the team is unit-first and the domain is well-understood -->

## Open questions
<!-- example: 2026-05-29T10:14:32Z — confirm the retention window with compliance before the next stage hardens the schema -->
- 2026-10-18T22:08:27Z [unit u1-core-loop-slice] — Session lifetime and the sign-in attempt limit (assumed 5 per 15 minutes) need confirming in NFR Requirements.
