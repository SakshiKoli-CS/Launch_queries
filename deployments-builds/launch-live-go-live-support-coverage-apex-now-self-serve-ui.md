QUESTION

Customer **UVA Health** requested **live Launch support on a go-live call** (cutover adding **4 new subdomains** + a few **edge function** changes). What's the process for arranging go-live coverage, and does a **Launch engineer need to join**?

_Keywords: live go-live support, cutover call coverage, Launch engineer join go-live, apex domain now self-serve UI, T2 support go-live, add subdomains edge function changes, schedule mutually convenient time, prerequisites checked engineering._

ANSWER

## Live go-live support is available — arrange coverage in advance

- Customers can request a **CS technical representative to join a go-live/cutover call** for real-time support (UVA had done this on a prior launch and found it beneficial).
- **Schedule at a mutually convenient time** where possible (time-zone overlap with the India team) — request rescheduling if the slot is very off-hours; otherwise engineering ensures **prerequisites are checked** so it can be handled independently.
- For this cutover, **T2 Support** attended the call and **Launch Engineering equipped them** with everything needed; a **Launch engineer can be on standby** rather than proactively joining when the work is self-serve.

## Key change — adding an Apex domain is now self-serve in the Launch UI

- **Previously**, adding an **Apex domain** required **manual steps and a Launch developer had to join** the go-live.
- **Now the feature is fully available in the Launch UI**, so a **Launch engineer no longer must join** just for apex setup — **T2 Support can handle the go-live**, with engineering available if an issue arises.
- (This particular cutover didn't even need an apex — just **4 new subdomains + edge function code changes**.)

## Generalized guidance (for future similar queries)

- **"Can we get live support for our go-live/cutover?"** → **Yes** — request coverage in advance; prefer a **mutually convenient time**. **T2 Support can run the call** with **Launch engineering on standby / prepping prerequisites**; a Launch engineer doesn't have to proactively join for now-self-serve tasks.
- **Adding an Apex domain is now self-serve in the Launch UI** (it used to require a Launch developer + manual steps) — so apex setup no longer mandates engineer attendance at go-live.
- Typical self-serve cutover work: **adding subdomains + edge-function changes** + domain redirects (old domains → new URL structure on the target site).
- Related: [cold start / health-check warming set at go-live](launch-npm-err-invalid-arg-type-path-null-security-fix-revert-sso-ssr-cache-cold-start.md), [cert pending → DCV + go-live support](../custom-domains-ssl/launch-cert-pending-dcv-needs-cname-not-txt-domain-limit-golive.md).

## Status

- **Resolved / go-live successful.** Coverage arranged with **T2 Support attending** + Launch engineering prepping prerequisites/standby; cutover (**4 subdomains + edge-function changes**) went **smoothly with no issues.** Reusable facts: **live go-live support is available (schedule mutually convenient)**; **Apex domain addition is now self-serve in the UI**, so an engineer need not proactively join.
