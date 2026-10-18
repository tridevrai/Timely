# Team Practices

> Integrated from the lead draft and the completed interview in
> `practices-discovery-questions.md`. This is a solo personal project: one
> person writes the code and approves every merge and release. The technology
> stack is not chosen yet, so tool names are left for the stage that decides it.

## Way of Working

- We use trunk-based development with short-lived branches. Each piece of work lives on its own branch for a day or two and then merges into `main`. We do not keep long-lived feature or release branches.
- CI runs on every branch, and a failing CI run blocks the merge into `main`. Branches exist so that this check has a merge to block.
- For Construction worktrees, the base branch is `main` and the merge target is `main`.
- We squash-merge each Bolt branch into `main`, so each Bolt becomes one commit on `main` named by the Bolt slug. The full Bolt commit history stays on the source branch until the worktree is discarded.
- If we need more than one environment, we keep one trunk and gate releases with tags or environment-specific deployment configuration, not with release branches.
- One person is both author and approver. Approving a merge or a release is a deliberate step by that person, not an automatic one.
- The workspace is not a git repository yet. The repository, its remote, and the hosting platform get set up before the first Construction work.

## Walking Skeleton

- We build a thin end-to-end slice first and the owner approves it before the rest of the features go in.
- The slice runs the product's core loop the whole way through: brain dump, then sorting with the 2-minute rule and Do / Delegate / Defer / Drop, then time-blocking the result on the calendar, then a Pomodoro focus session.
- Why: the slice proves the core idea works before any extras get built, and it keeps us from "productivity theater", meaning settings and polish built ahead of the core loop.
- The first Unit is the smallest working integrated slice. It goes through every applicable per-unit stage, including Code Generation, before later Units. It is verified with the intent's recorded, human-approved Construction Verification Command, which must show the real end-to-end result, and then the owner approves the skeleton checkpoint.
- Settings, polish, and secondary features wait until after the skeleton is approved.

## Testing Posture

- **Methodology**: test-after
- **Methodology evidence**: The owner chose this at the interview (Question 3, option A). It matches the `classic` scope default, and there is no existing code or test history pointing to another cadence.
- **Ordering**: Implement each applicable testable layer, then write and run that layer's tests.
- Tests are a first-class deliverable in every Bolt; a Bolt is not done without its tests.
- Coverage floor: at least 80% line coverage, and the suite runs in CI before merge (Question 4, option A). A merge into `main` is blocked when tests fail or coverage drops below the floor.
- We never lower the coverage floor or disable a failing test to make a check pass (see `discovered-rules.md`).
- The active Test Strategy still decides test volume and test types; the coverage floor adds to it and never replaces it.
- The test framework, coverage tool, and CI platform are chosen together with the technology stack.

## Deployment

- We deploy to a staging environment automatically on every merge to `main`.
- Production deploys need a separate manual approval. On this solo project, the owner is the approver.
- We do not deploy every merge straight to production. Continuous deployment to production may come later, as an explicit decision once test coverage and monitoring justify it.
- No staging or production environment exists yet. The hosting target and the CD platform are chosen when the technology stack is decided.

## Code Style

- We use the standard formatter and linter for the chosen language and keep their configuration in the repository root.
- The formatter and linter run in CI, and a lint failure blocks the merge into `main`.
- Naming follows the chosen language's usual conventions. We have no project-wide naming rules beyond that.
- Agents read the project's linter configuration first and only suggest a style change when the linter does not already cover it.
- No language, formatter, or linter is chosen yet. The specific tools are picked when the technology stack is decided.
