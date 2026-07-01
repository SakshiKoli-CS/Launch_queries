QUESTION

Customer **Dash In** is migrating **3 URLs** — `dashin.com`, `locations.dashin.com`, `dashin.com/careers/` — all **served by the same stack** on Launch. Three questions:
1. **How do we route the request URL to specific data entries per domain** (different domains → different content, from one stack)?
2. How do we **set up the DNS switch / go-live** for these (mix of **apex** `dashin.com` and **subdomain** `locations.dashin.com`)?
3. How do we **implement redirects** (e.g. to location pages) on Launch?

_Keywords: multiple domains single stack, route request URL to specific data entries per domain, per-domain content routing, apex vs subdomain go-live, apex domain manual setup Launch team, add subdomain www Launch UI CNAME, TXT records validate hostname issue certs before DNS switch zero downtime, edge redirects documentation example, locations subdomain, careers path._

ANSWER

## Q1 — Routing per-domain content from one stack → clarify the requirement (edge-function/rewrite territory)

- **One stack can serve multiple domains**, and routing **different domains/paths to different content** is done at the **edge** (Edge Functions / rewrites) by **inspecting the incoming host/path** and mapping it to the right data — but the **exact requirement needs clarification** (e.g. does each domain map to a distinct content-type/locale/folder, or is it path-based?). Get the customer to specify the mapping before prescribing a pattern.
- General approach: an **Edge Function reads the request host** (`locations.dashin.com` vs `dashin.com`) and/or path (`/careers/`) and **rewrites/fetches the corresponding entries**. See the multi-stack/multi-domain and rewrites related entries.

## Q2 — DNS switch / go-live: subdomains are self-serve, apex needs the Launch team

- **Subdomains** (`www.dashin.com`, `locations.dashin.com`): the customer **adds them in the Launch UI** and sets the **CNAME** shown in the UI at their DNS provider — **self-serve**.
- **Apex** (`dashin.com`): **apex setup is done manually by the Launch team** (apex can't be a CNAME) — Launch **joins the go-live call** to configure it.
- **Zero-downtime cutover for an existing live site:** customer **adds the subdomains in the Launch UI and notifies Launch → Launch provides the TXT records to validate hostnames + issue certs BEFORE the DNS change** → only then does the customer update the DNS CNAME records. (Pre-provision certs so the flip is instant.)

## Q3 — Redirects → Edge Redirects

- **Redirects** (e.g. to specific location pages) are handled via **Edge Redirects**. Refer to the **Edge Redirect documentation** and the **Edge Redirect Launch example**. (For anything needing query-string preservation or dynamic logic, use an **Edge Function** — see related.)

## Generalized guidance (for future similar queries)

- **Multiple domains on ONE stack, different content per domain** → serve all domains from the stack and **route by host/path at the edge** (Edge Function/rewrite). **Always clarify the exact per-domain → content mapping** before prescribing the implementation.
- **Go-live DNS:** **subdomains = self-serve** (add in Launch UI → set CNAME); **apex = manual, Launch-team-assisted** on the go-live call (apex can't CNAME). For **live sites, pre-provision certs** (add subdomain → notify → Launch gives TXT → validate/issue → then flip CNAME) for **zero downtime**.
- **Redirects → Edge Redirects** (docs + example); dynamic/query-preserving redirects → **Edge Function**.
- Related: [multi-stack / multiple domains on a single Launch project](multi-stack-domains-single-launch-project.md), [multi-domain go-live full process (www Cloudflare / apex Fastly)](launch-multi-domain-golive-apex-fastly-www-cloudflare-full-cutover-process-dns-records-bfs.md), [go-live support request in advance + standard steps](launch-golive-support-request-in-advance-standard-steps-schedule-weekday-ist-pods.md), [launch.json redirect query string not preserved → Edge Function](../edge-functions-rewrites/launch-json-redirect-query-string-utm-not-preserved-question-mark-deploy-fail-use-edge-function.md), [phased migration rewrites to legacy backend](../edge-functions-rewrites/launch-phased-migration-rewrites-legacy-backend.md).

## Status

- **Guided (routing question pending clarification).** **Go-live:** add **subdomains in the Launch UI (self-serve CNAME)**; **apex (`dashin.com`) set up manually by the Launch team** on the go-live call; for the **existing live site**, **pre-provision certs** (add subdomain → notify → TXT validation/issue → then flip CNAME) for **zero downtime**. **Redirects → Edge Redirects** (docs + example). **Per-domain content routing** needs the customer to **clarify the exact mapping** before recommending an edge pattern.
