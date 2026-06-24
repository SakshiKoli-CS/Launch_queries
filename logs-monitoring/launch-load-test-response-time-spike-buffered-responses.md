QUESTION

Customer **Philips** ran a **load test** (20 May 2026, 22:13–23:14 IST) on their **ACC environment** and observed a **significant response-time spike ~33 minutes in (~22:46 IST)**, with **average response times reaching ~50 seconds** across multiple transactions. They asked whether there was any **server-side behavior during that window — autoscaling events or infrastructure restarts** — that would explain it. (They also planned a second run to see if the spike was a one-off.)

_Keywords: load test, response time spike, high latency, 50 second response, autoscaling event, infrastructure restart, buffered response, request duration, performance test, application-side latency, ACC environment._

ANSWER

## Findings

1. **Buffered responses (platform-side, fixed):** Some requests had responses **buffered for longer than expected** during the reported window, which **contributed to the increased request durations**. **This behavior has been fixed** — the customer was asked to **re-run the load test** to confirm.
2. **Application-side processing:** For some requests, a **significant portion of the total response time was spent within the application itself**. The customer was asked to **review application-side processing** as a likely contributor to the observed latency.
3. **No autoscaling / restart issue:** **No problems with autoscaling or infrastructure restarts** were observed during the same timeframe — so the spike was **not** caused by scaling events or restarts.

## How to investigate this class of report

- Need from the customer: the **actual URL/domain** used for the test and the **environment UID** (environment name + org is enough to look up the UID).
- On the Launch side: review **request durations** in the window for **response buffering** vs **time spent in the application** — splitting platform time from app time is the key diagnostic.
- Rule out **autoscaling/restart** events explicitly (here, there were none).

## Generalized guidance (for future similar queries)

- **"Big latency spike during a load test — was it autoscaling/restarts?"** → Check **autoscaling/restart events first** (often *not* the cause), then split the latency into **platform-side (e.g. response buffering)** vs **application-side processing**.
- **Response buffering** taking longer than expected is a platform-side contributor that can be fixed; ask the customer to **re-test** after a fix.
- **App-side time** is the customer's to optimize — if a large share of total response time is inside the app, point them to **review their application processing**, not just the platform.
- Always get the **exact URL + environment UID** to scope the investigation to the right environment.

## Status

- Buffered-response behavior **fixed**; customer asked to **re-run the load test** and to **review application-side processing**. No autoscaling/restart issues found. Awaiting re-test results / customer response at last update.
