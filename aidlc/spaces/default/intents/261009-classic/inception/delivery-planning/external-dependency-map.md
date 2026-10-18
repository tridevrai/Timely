# External Dependency Map: Timely

Things outside the code that a **Bolt** (one build pass over one unit, ending in
a working app) needs before it can finish. The owner handles all of them; the
AI builds the code.

| Dependency | Owner | Lead time | Needed by | If it slips |
|---|---|---|---|---|
| Git repository with `main`, plus a CI service | owner (AI can scaffold the CI config) | under 1 hour | B1 start | B1 can be built locally but can't be merged per team.md until CI exists |
| Self-hosted server with a domain name and HTTPS [Q6] | owner | hours to a day | B1 end-to-end check | B1's code and automated tests finish locally; the skeleton checkpoint waits for the iPhone check over HTTPS |
| iPhone home-screen install and notification permission (iOS 16.4+) | owner | minutes | B1 end-to-end check | background notifications can't be verified; in-app fallback only |
| Web push signing keys, stored as environment variables | owner (AI documents the generation step) | minutes | B1 | notifications can't be sent |
| Google Cloud project, Calendar API enabled, OAuth consent screen in testing status with the owner as a test user [Q5] | owner | under 1 hour | B4 start | B4's code is built and tested against the fake adapter; the manual real-calendar check waits |
| Google app verification (optional, removes the weekly reconnect) [Q5] | owner | days to weeks | none (later improvement) | keep reconnecting weekly |
| Database backups, at least daily (requirements NFR3) | owner, on the chosen host | hours | before relying on the app daily (end of B1) | risk of data loss |

No other team, partner API or approval is involved.
