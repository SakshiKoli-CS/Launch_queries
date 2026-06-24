QUESTION

Customer **Milestone Systems** (Azure EU) reported **cold starts after only ~20 minutes of inactivity**. The SE's understanding was that **cold starts should only happen after longer idle periods** — so is a 20-minute cold start unexpected, and what's the right way to handle it?

_Keywords: cold start, cold starts after inactivity, container idle termination, server not staying warm, keep warm, Pingdom check, Azure idle 5 minutes, non-production intermittent traffic, instance restart, scale to zero._

ANSWER

## Expected behavior — idle containers are terminated quickly

- On **Launch (Azure)**, any container that stays **idle for ~5 minutes is automatically terminated**. So a cold start after a period of inactivity is **expected**, especially in **non-production** environments where traffic is intermittent (dev/testing).
- This means cold starts can occur **far sooner than "long" idle periods** — the ~5-minute idle-termination is the relevant threshold, not hours.

## Keeping the server warm (for production / low-traffic sites)

- For a **production site with intermittent traffic**, keep at least one container active with a **Pingdom (uptime) check** that periodically hits the site.
- This keeps a container **warm even during low/no traffic**. Launch still **autoscales** instances up/down based on traffic (which can *look* like frequent restarts), but the site won't sit idle long enough to cold-start on real visits.
- **Recommendation:** it's better for the **customer to set up the Pingdom check themselves** than for support to configure it on their behalf.

## Scoping note

- Confirm the **exact domain/environment** where the cold start was observed — a steady-traffic production domain (e.g. `customer.milestonesys.com` was receiving steady traffic) shouldn't cold-start often, so identify whether the report is on a **non-prod / low-traffic** environment.

## Generalized guidance (for future similar queries)

- **"Cold starts after only N minutes idle — is that a bug?"** → No. Launch (Azure) **terminates idle containers after ~5 minutes**, so cold starts after short inactivity are **expected**, particularly in **non-production / intermittent-traffic** environments.
- **To avoid cold starts on a low-traffic production site:** use an **external uptime check (e.g. Pingdom)** to keep a container warm; Launch continues to autoscale normally.
- **Confirm the environment first** — frequent cold starts on a genuinely steady-traffic production site would warrant a closer look; on dev/preview it's normal.
- Prefer guiding the **customer to own the keep-warm check** rather than support configuring it.

## Status

- Confirmed the behavior is **expected** (idle-termination ~5 min); recommended a **customer-managed Pingdom check** to keep production warm; asked the customer to confirm the specific environment/domain.
