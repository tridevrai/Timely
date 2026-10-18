# Evidence

## Participants

Lead only (`aidlc-pipeline-deploy-agent`). Collaborators are off for this `classic` run, so no support agent reviewed the draft and there are no contribution files. The lead wrote the draft (Step 2), the human answered the interview (Step 4), and the lead integrated the result (Step 5).

## What was inspected

- Project type and scope from the dispatch brief: greenfield, scope `classic`, depth Standard, workspace `/Users/sage/Documents/PersonalProjects/timely`.
- Workspace state: not a git repository, no application code, only the AI-DLC shell. There was no git history, CI configuration, deployment configuration, or lint/format configuration to inspect.
- `aidlc/spaces/default/memory/org.md`, the five matching sections (Way of Working, Walking Skeleton, Testing Posture, Deployment, Code Style). Used as the suggested answers in the interview, not as team facts.
- `aidlc/spaces/default/memory/team.md`: every section empty, so there was no affirmed baseline and this is a first run, not a re-run.
- `.claude/scopes/aidlc-classic.md`: the scope declares `skeleton: off` (relevant to the Walking Skeleton note below).
- `.claude/aidlc-common/stages/inception/practices-discovery.md` for the step and artifact contract.
- `practices-discovery-questions.md`: the completed interview, 7 questions and 2 follow-ups.
- Brownfield upstream artifacts (code-structure, technology-stack, dependencies, code-quality-assessment, architecture, business-overview): not applicable, the project is greenfield.

## What was inferred

- Nothing about the team was inferred from code or history; none exists. Every practice in `team-practices.md` comes from an interview answer or, where noted below, from an `org.md` default the owner did not re-decide.
- The project is a solo personal project. This comes from the owner's own reasoning in the Question 1 answer ("a personal, single-user project").

## Interview decisions

| Area | Question | Answer | Result |
|---|---|---|---|
| Way of Working | Q1 | C: work directly on `main`, no feature branches ("this is a personal, single-user project ... feature branches add ceremony without benefit") | Replaced by Follow-up 1 |
| Way of Working | Follow-up 1 | C: switch to short-lived branches after all, so CI can block a merge into `main` | Affirmed: trunk-based, short-lived branches (1-2 days) merged into `main`, CI blocks the merge |
| Walking Skeleton | Q2 | A: build the thin end-to-end slice first and have the owner approve it before the rest | Affirmed: skeleton on, through the core loop (brain dump, 2-minute rule / Do-Delegate-Defer-Drop sort, calendar time-blocking, Pomodoro focus session); reasoning kept: avoid "productivity theater" |
| Testing Posture | Q3 | A: test-after | Methodology `test-after`; Ordering: implement each applicable testable layer, then write and run that layer's tests |
| Testing Posture | Q4 | A: at least 80% line coverage, tests run in CI before merge | Affirmed coverage floor and CI gate |
| Deployment | Q5 | A: staging on every merge, production after a separate manual approval | Affirmed; the approver is the solo owner |
| Code Style | Q6 | A: standard formatter and linter for the chosen stack, run in CI, lint failure blocks merge | Affirmed; tools chosen with the stack |
| Hard rules | Q7 | X: "Best practices" | Too broad to bind; clarified by Follow-up 2 |
| Hard rules | Follow-up 2 | A, B, C, D | Four rules in `discovered-rules.md` (below) |

### How the Question 1 conflict was resolved

Question 1 chose working directly on `main`. Questions 4, 5 and 6 then chose options that rely on a merge step tests and lint can block. With no branches there is no merge to block, so Follow-up 1 asked how checks should run. The owner chose to switch to short-lived branches. The Follow-up 1 answer supersedes Question 1, and `team-practices.md` records the branch-based workflow only.

### Carried from the org default, not re-chosen

- **Squash-merge.** The owner chose short-lived branches in Follow-up 1 but did not separately choose between squash-merging and keeping each branch's full history (options A and B of Question 1 were not re-asked). `team-practices.md` keeps the `org.md` default: each Bolt is squash-merged into one commit on `main`. The owner can change this at the affirmation gate.
- **Base and target `main` for Construction worktrees**, and **one trunk with tags or environment config instead of release branches**: `org.md` defaults, consistent with the chosen workflow and not contradicted by any answer.
- **Idiomatic naming** and **agents read the linter configuration first**: `org.md` Code Style defaults, consistent with the Question 6 answer.

### Hard rules (Follow-up 2)

The owner picked all four options, so all four are human-stated and binding:

- A: NEVER commit secrets or credentials; keep them in environment variables or a secrets manager. (Forbidden)
- B: NEVER lower the coverage floor or disable a failing test to make a check pass. (Forbidden)
- C: NEVER add a dependency without pinning its version in a lockfile. (Forbidden)
- D: ALWAYS validate input at system boundaries (forms, APIs, imported files). (Mandated)

No other rules were added. The Question 7 answer "Best practices" was not turned into any further rule.

## Remaining uncertainty

1. **Walking skeleton and the scope default.** The `classic` scope declares `skeleton: off`, and `org.md` says to select the walking skeleton only when the active scope declares `skeleton: on`. The owner explicitly chose to build the slice first. Once promoted, the team's affirmed stance in `team.md` is on. The conductor should confirm at Construction entry that the skeleton checkpoint will actually run for this intent and not be skipped because of the scope default.
2. **Squash vs keep history.** Carried from the org default (see above), not explicitly re-chosen.
3. **Technology stack and tools.** Language, framework, formatter, linter, test framework, coverage tool, CI platform, hosting target, and CD platform are all undecided. They are expected from a later stage.
4. **Repository and environments.** No git repository, remote, staging environment, or production environment exists yet. They need to exist before Construction can merge, run CI, or deploy.
