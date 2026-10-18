# Delivery Planning Questions

This step decides the order in which the five units get built. Each one is
built as a **Bolt**: one build pass over a piece of the work, ending in
something that runs and that you approve. The dependency map from Units
Generation fixes one thing already: **U1 Core loop slice comes first**. It is
your walking skeleton, the thin end-to-end slice you approve before anything
else. After U1, U2, U3 and U4 can come in any order; U5 needs U3.

## Question 1: After the first slice, what should come next?

A. Most useful first: U2 Sort and lists → U3 Day and focus → U4 Calendar sync → U5 Reminders and insights. You get a complete daily loop early; Google sync arrives fourth (suggested)
B. Riskiest first: U4 Calendar sync straight after U1 (it depends on Google and is the hardest part), then U2 → U3 → U5
C. A mix: U2 first for daily use, then U4 for the risk, then U3 → U5
X. Other (please specify)

[Answer]: A

## Question 2: Should the order be scored with a formal model?

Context: WSJF ("weighted shortest job first") scores each piece by value,
urgency and risk reduction divided by size, and builds the highest score first.

A. No: with five units, a written reason for the order is enough (suggested)
B. Yes: score each unit with WSJF and follow the scores
X. Other (please specify)

[Answer]: A

## Question 3: How big should each Bolt be?

A. One unit per Bolt, five Bolts in total, each ending in a working, approved app (suggested)
B. Bundle smaller units together (for example U2 + U3 in one Bolt)
X. Other (please specify)

[Answer]: A

## Question 4: Can Bolts be built at the same time?

A. No: one after another, each approved before the next starts. This fits a solo project and the recorded setting (suggested)
B. Yes: after U1, build U2, U3 and U4 side by side
X. Other (please specify)

[Answer]: A

## Question 5: Google Calendar needs a Google Cloud project with an approved sign-in screen. How should we handle that?

Context: Google sign-in apps left in "testing" status issue access that expires
after about 7 days, so you'd have to reconnect weekly. Publishing a personal
app that only uses calendar events usually needs Google's verification, which
can take days to weeks.

A. I'll create the Google Cloud project before U4 starts. Keep the app in testing for now and accept reconnecting weekly; consider verification later (suggested)
B. I'll start Google verification now so it's ready by the time U4 is built
C. Not sure; flag it as a risk and decide when U4 comes up
X. Other (please specify)

[Answer]: A

## Question 6: Where will the server run, and who sets it up?

Context: Timely is self-hosted. Building and testing work locally, but your
iPhone needs the app reachable over HTTPS (for the home-screen install and
notifications).

A. I have, or will set up, a server with a domain name and HTTPS before U1's end-to-end check; the exact host is chosen in Infrastructure Design (suggested)
B. Run it on my Mac only for now and reach it from my phone on the home network; real hosting later
C. Not decided; flag it as a risk
X. Other (please specify)

[Answer]: A

## Question 7: What worries you most about this build?

A. Timers and notifications not being reliable on iPhone
B. Google Calendar sync creating duplicates or losing changes
C. Building too much before using it (productivity theater)
D. Nothing in particular
X. Other (please specify)

[Answer]: C
