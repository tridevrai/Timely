# Risk and Sequencing Rationale: Timely

## Heuristic

**Walking skeleton first, then value first** [Q1]. The **walking skeleton**
(Cockburn, *Crystal Clear*) is the thinnest version that runs end to end, built
first to prove the pieces connect. After it, Bolts (one build pass per unit)
ship in the order that makes the app most useful soonest. No formal scoring
model such as WSJF (weighted shortest job first) is used: with five units and
one person, a written argument is clearer [Q2].

## Order and why

1. **B1 Core loop (U1)**: required first by the team's walking-skeleton practice
   and by the dependency map (every other unit depends on U1). It proves the
   riskiest architectural parts early: background notifications on iPhone, the
   timer design and the single-app structure.
2. **B2 Sort and lists (U2)**: after B1, Delegate, Defer and Drop are what turn
   Timely from a demo into the owner's real daily tool. It is the most useful
   next step.
3. **B3 Day and focus (U3)**: makes the day and the timers dependable for a full
   working day and adds the end-of-block decisions, which B5 needs.
4. **B4 Calendar sync (U4)**: the hardest and most external piece. It goes fourth
   because the owner chose usefulness over risk [Q1]; the app is fully usable
   without it, and the delay gives time to set up the Google Cloud project [Q5].
5. **B5 Reminders and insights (U5)**: supporting habits. It needs U3, and its
   summary only means something after weeks of real use.

**Deviation from the dependency map**: none. This is a valid topological order.

## Main worry: building too much before using it

The owner's biggest concern is "productivity theater", spending effort on
features before the core is used [Q7]. Countermeasures:

- B1 is the smallest slice that's useful on its own.
- Every Bolt's Definition of Done includes at least one real day of use before
  the next Bolt starts (`bolt-plan.md`).
- Each finished Bolt is a decision point: if the app is already enough, later
  Bolts are dropped or reordered rather than built by default.
- Version 1 has no themes, tags or recurring tasks (requirements Q14).

## Risk register

| Risk | Likelihood | Impact | Mitigation | Bolt |
|---|---|---|---|---|
| Background notifications unreliable on iPhone (web push needs a home-screen install and iOS 16.4+) | Medium | High | Proven in B1 on a real iPhone; the in-app end-of-block prompt is a fallback (US7.1 AC3) | B1 |
| No HTTPS server ready for B1's end-to-end check | Medium | High | Owner sets it up before B1's check [Q6]; host chosen in Infrastructure Design | B1 |
| Timer drift after sleep or reload | Low | Medium | Time is computed from recorded start instants (ADR-002); manual 10-minute sleep test in B3 | B1, B3 |
| Google testing-status access expires about weekly | High | Low | Accepted [Q5]; reconnect prompt (US8.4) makes it quick; verification later if annoying | B4 |
| Google sync duplicates or loses changes | Medium | High | Idempotent writes keyed by block, outbox delivery, fake-adapter tests, manual real-calendar check | B4 |
| Clock skew between server and Google affects "newest change wins" | Low | Medium | Tolerance set in NFR Design (contract open question) | B4 |
| Scope creep / productivity theater | Medium | High | Use-it gate between Bolts; drop later Bolts if not needed | all |
| Tech stack not yet chosen | Certain | Medium | Chosen in B1's Construction design stages; formatter, linter and test tools follow (team.md Code Style) | B1 |
