QUESTION

The Launch-hosted website **www.microstrategy.com** went DOWN with the error **"Request URI does not exist"**, redirecting to **https://www.strategy.com/software**. A monitoring alert triggered on **June 18, 2026 at 23:20 IST** and the issue was ongoing for ~20 minutes at report time. The site had been healthy with **no outages in the prior 2 months**; the last successful ping was **June 18, 23:15:49 IST** (monitors ping every 5 min from NA and EU regions). Was this intended behavior (ongoing deployment / customer-side change) or a platform issue?

- **Org:** Microstrategy — `blt45a2a52367b47001`
- **DC:** AWS NA

ANSWER

**Symptoms**
- `www.microstrategy.com` returned **"Request URI does not exist"** and redirected to `https://www.strategy.com/software`.
- `https://www.strategy.com/` (the rebranded domain) was working fine throughout.
- Monitor pinged every 5 min from NA + EU; last good ping **23:15:49 IST**, 200s before that.

**Investigation**
- Launch logs showed **404s increasing around 21:20 IST**, followed by a spike around **23:20 IST**. Exact start was not conclusive between the two timestamps.
- **Deployments from the customer's end** occurred around the same timeframes.
- Initial assessment: most likely an **application/code change introduced during the customer's deployment**, not a platform fault.

**Cause / hypotheses**
- MicroStrategy rebranded to **Strategy** (announced Feb 5, 2025) and migrated to the `strategy.com` domain months ago.
- Likely the **pre-existing redirect rules / mapping for `microstrategy.com` were deprecated or inadvertently removed** during a recent deployment, breaking the redirect.

**Resolution**
- Customer was notified via support communication **00059565**.
- Shortly after, `https://www.strategy.com/software` began working and the **monitor returned 200 (Success)** again. Issue self-resolved on the customer side.

**Follow-up / suggested checks**
- Monitor was **temporarily suspended** during investigation.
- Open question for the customer: confirm whether the monitored target should be **updated to `www.strategy.com`** given the rebrand (to be confirmed by the team).

**Notes**
- Platform-side: no Launch infrastructure issue identified; behavior traced to customer deployment + redirect mapping.
