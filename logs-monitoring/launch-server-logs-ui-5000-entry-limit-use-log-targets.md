QUESTION

Customer **Eight25Media** (SF case **00053919**) is using **Launch → Server Logs** to retrieve the **last 24 hours of server logs** for debugging/monitoring, but the **UI shows a limit-related message** that prevents them from seeing the full log window they want.

Is there a **retention or retrieval limit** for server logs in Launch, and can the **5,000-log display limit** be increased?

_Keywords: server logs limit, Launch UI 5000 logs, log retention retrieval limit, only most recent 5000 entries, last 24 hours logs, increase log limit not configurable, Log Targets external logging, forward logs._

ANSWER

## The 5,000-entry UI limit is expected and not configurable

- The Launch UI is designed to display the **most recent 5,000 log entries (≈ the last 24 hours)** to keep the experience **fast and responsive.**
- This is **expected behavior** and is **not configurable within the UI** — the 5,000 cap **cannot be increased**.

## For extended history / deeper analysis — use Log Targets

- For **longer retention or deeper analysis**, Launch offers **Log Targets**, which **forward logs to an external logging system.**
- This gives **full flexibility to store, retain, and search** logs per the customer's own requirements (beyond the 5k UI window).
- Refer to the **Log Targets documentation** for setup and configuration.

## Generalized guidance (for future similar queries)

- **"Can we see more than 5,000 server logs / increase the UI log limit?"** → **No** — the UI intentionally shows only the **most recent ~5,000 entries (~24h)** for performance; it's **not configurable.**
- **Position to the customer:** the UI is for **recent/quick** viewing; for **retention, history, and search at scale**, set up **Log Targets** to forward logs to an external system. That's the supported path for full log access.

## Status

- **Answered.** The **5,000-entry UI limit is expected and non-configurable**; recommended **Log Targets** for extended retention/analysis. Customer satisfied.
