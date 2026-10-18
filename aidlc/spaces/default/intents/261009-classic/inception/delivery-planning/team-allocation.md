# Team Allocation: Timely

A **Bolt** is one build pass over one unit of work, ending in a working app the
owner approves. A **mob** is the group that builds a Bolt together.

Team Formation was skipped (the classic plan has no Ideation phase), and this is
a solo personal project. So:

| Bolt | Unit | Built by | Approved by |
|---|---|---|---|
| B1 Core loop | U1 | aidlc-developer-agent (AI), run from this session | the owner |
| B2 Sort and lists | U2 | aidlc-developer-agent (AI) | the owner |
| B3 Day and focus | U3 | aidlc-developer-agent (AI) | the owner |
| B4 Calendar sync | U4 | aidlc-developer-agent (AI) | the owner |
| B5 Reminders and insights | U5 | aidlc-developer-agent (AI) | the owner |

- One Bolt at a time, each approved before the next starts [Q4]. There is no
  Program Board (a cross-team view of who builds what when), because there is
  only one team.
- The owner is the only approver for merges, the skeleton checkpoint and
  production releases (team.md Way of Working and Deployment).
- Owner tasks outside the AI's reach: the server, domain and HTTPS before B1's
  end-to-end check [Q6]; the Google Cloud project before B4 [Q5]; the iPhone
  home-screen install and notification permission during B1. See
  `external-dependency-map.md`.
