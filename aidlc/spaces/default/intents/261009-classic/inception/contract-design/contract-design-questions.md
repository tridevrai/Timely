# Contract Design Questions

This step pins down exactly what crosses each boundary: what's sent, in what
shape, and what happens when it fails.

Timely is one app [units Q1], so the boundaries that matter are:
- **The web app ↔ the server**: the main contract. The browser on your Mac or iPhone is the outside consumer.
- **Building blocks inside the server**: module interfaces and internal "something changed" events (ADR-010), for example Tasks telling Planning a task left Do Now.
- **The server ↔ Google Calendar and the push-notification service**: their formats are fixed by them; we only decide how we use them.

## Question 1: How should the web app talk to the server?

A. A JSON API over HTTPS with resource-style addresses (for example `POST /api/v1/tasks`, `PATCH /api/v1/blocks/{id}`) (suggested)
B. GraphQL, with one endpoint and flexible queries
C. Remote procedure calls (one call per action, for example `sortTask`)
X. Other (please specify)

[Answer]: A

## Question 2: How should the web app sign in to the API?

A. A secure, http-only session cookie set at sign-in, plus protection against cross-site request forgery. The browser handles it automatically, and scripts can't read it (suggested)
B. A bearer token that the web app stores and sends with each request
X. Other (please specify)

[Answer]: A

## Question 3: How should an open screen learn about changes made on your other device, or by calendar sync?

A. The app re-fetches when you return to it, and every 60 seconds while it's open (suggested; simple and enough for one person)
B. The server pushes changes live over a persistent connection
X. Other (please specify)

[Answer]: A

## Question 4: How should Timely pick up changes made in Google Calendar?

Context: the requirements say Google-side changes show up within 5 minutes.

A. Ask Google for "what changed since last time" every 2 minutes, and immediately when you open Today (suggested; works even if the server isn't reachable from the internet)
B. Have Google notify the server when something changes (needs the server reachable from the internet with HTTPS), with a slow fallback check
X. Other (please specify)

[Answer]: A

## Question 5: What should happen when an edit is made against an out-of-date copy (edited on the other device first)?

A. The server refuses with a "conflict" response that includes the current version, and the app asks you which version to keep (suggested; matches the approved stories)
B. The newest save always wins silently
X. Other (please specify)

[Answer]: A

## Question 6: How reliable must the internal "something changed" events be?

Context: for example, if the server restarts right after you move a block, the
calendar sync must still learn about the move.

A. Stored with the change in the same database save and processed afterwards, so no event is ever lost, even across a crash (suggested)
B. Handled in memory right after the save; an event can be lost if the server crashes at that moment
X. Other (please specify)

[Answer]: A

## Question 7: How should the API be versioned?

A. Every address starts with `/api/v1`. The web app is always shipped with the server, so they change together; a `v2` is only added if an outside client ever needs the old shape (suggested)
B. No version in the address
X. Other (please specify)

[Answer]: A
