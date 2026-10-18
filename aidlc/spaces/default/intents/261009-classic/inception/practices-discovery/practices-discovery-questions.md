# Practices Discovery Questions

This is a new project with no code or git history, so there is nothing to infer
from. These questions fix how you want to work. Each option marked "(suggested)"
is the team default from `aidlc/spaces/default/memory/org.md`; pick it, pick
another, or write your own under X.

## Question 1: How should finished work get into the main line of code?

Context: the folder is not a git repository yet. This decides the branching and
merge habits that every later build step follows.

A. Short-lived branches (a day or two) merged into `main`, with each unit of work squashed into one commit on `main` (suggested)
B. Short-lived branches merged into `main`, keeping the full commit history of each branch
C. Work directly on `main` with no feature branches (fits a solo project)
D. Longer-lived feature or release branches
X. Other (please specify)

[Answer]: C — this is a personal, single-user project (the handover document is written for one person's own workflow), so feature branches add ceremony without benefit.

## Question 2: Build a thin end-to-end slice first?

A walking skeleton is a minimal version that runs the whole way through, built
first to prove the pieces connect before the real features go in.

A. Yes, build the thin end-to-end slice first and have me approve it before the rest
B. No, build the features in order without a separate first slice
X. Other (please specify)

[Answer]: A — the product has one core loop (brain dump → 2-minute rule / Do-Delegate-Defer-Drop sort → time-block on the calendar → Pomodoro focus session). A thin slice through that loop proves the idea before adding extras, and it follows the document's warning against "productivity theater" (building settings and polish instead of the core).

## Question 3: In what order should code and tests be written?

Context: this sets the testing approach the build step follows. It is recorded as
a fixed methodology plus a one-sentence ordering.

A. Write each layer of code, then write and run its tests (test-after) (suggested)
B. Write a failing test first, then the code that makes it pass (test-driven)
C. Write behavior scenarios first, then the code, with lower-level unit tests after the code (a mix)
D. Write acceptance tests for each feature first, then build to satisfy them
X. Other (please specify)

[Answer]: A

## Question 4: How much automated test coverage should be required before code merges?

Context: this workflow type already requires at least 80% line coverage with the
tests running in CI before merge. You can keep that or raise it; it cannot be lowered.

A. At least 80% line coverage, tests run in CI before merge (suggested)
B. At least 90% line coverage, tests run in CI before merge
X. Other (please specify)

[Answer]: A

## Question 5: How should releases reach your environments?

Context: no staging or production environment exists yet. Production approvals
could be just you, if this is a solo project.

A. Deploy to a staging environment on every merge; production only after a separate manual approval (suggested)
B. Deploy every merge straight to production automatically
C. No deployment environments yet; decide later when there is something to deploy
D. Manual deploys only, whenever I decide
X. Other (please specify)

[Answer]: A

## Question 6: How should code formatting and style be enforced?

Context: the language and tools are not chosen yet, so this fixes the principle
and the tools get picked when the technology stack is decided.

A. Use the standard formatter and linter for the chosen language, run them in CI, and block merges on lint failures (suggested)
B. Use the standard formatter and linter, but only as local checks, not blocking merges
C. I will provide my own style guide that overrides the language defaults
X. Other (please specify)

[Answer]: A — use the standard tools for the chosen stack

## Question 7: Are there any hard rules every part of the project must follow or never do?

Context: examples are "never commit secrets", "always use a specific license",
or "never add a dependency without review". Only rules you state become binding.

A. No hard rules for now
B. Yes, I have some (describe them under X)
X. Other (please specify)

[Answer]: X - Best practices

## Follow-up 1: With no feature branches, when should tests and checks run?

Context: Question 1 chose working directly on `main`, but Questions 4, 5 and 6
assumed a merge step that tests and lint can block. Without branches there is no
merge to block, so the checks need a different trigger.

A. Run tests, coverage and lint locally before every push (a pre-push hook), and again in CI on every push to `main`; staging deploys only when that CI run is green
B. Run them only in CI on every push to `main`; a red run blocks the staging deploy and must be fixed next
C. Switch to short-lived branches after all, so CI can block a merge into `main`
X. Other (please specify)

[Answer]: C

## Follow-up 2: Which "best practices" should be binding rules?

Context: Question 7 answered "best practices". Only rules you pick here become
hard constraints that every later step must follow.

A. Never commit secrets or credentials; keep them in environment variables or a secrets manager
B. Never lower the coverage floor or disable a failing test to make a check pass
C. Never add a dependency without pinning its version in a lockfile
D. Always validate input at system boundaries (forms, APIs, imported files)
X. Other (please specify)

[Answer]: A, B, C, D
