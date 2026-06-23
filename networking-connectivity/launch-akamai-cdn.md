QUESTION

A customer (Breguet, via Valtech/partner) asked whether **Launch is compatible with Akamai CDN**, given Breguet's mandatory security and delivery stack. Akamai is non-negotiable for the customer due to security requirements and includes the following modules: Advanced Delivery, EdgeWorkers, Image and Video Manager, Visitor Prioritization Cloudlet, China CDN, DoS Protection, WAF, Client Reputation, Bot Manager Premier, and Malware Protection.

The secondary issue that surfaced during integration testing: a **build failure** traced to a misconfigured **`CONTENTSTACK_REGION` environment variable**, where using `'NA'` instead of `'us'` caused the Contentstack SDK to construct an invalid CDN hostname. After fixing the region value, the site returned a **CF001 Internal Server Error** (unresolved at end of thread).

ANSWER

**Akamai / external CDN compatibility**

- **Launch can be made to work with Akamai** (or any external CDN placed in front of Launch's own CDN layer).
- However, placing an external CDN in front of Launch introduces **caveats** the customer must be aware of—these are the same considerations that apply when using Cloudflare or any other CDN in front of Launch. The internal reference document covering this is titled *"Using Launch with another CDN (like Cloudflare) in Launch"*.
- Launch does not natively provide the same feature set as Akamai's security modules (WAF, Bot Manager Premier, DDoS protection, etc.). The recommendation is to keep Akamai in the stack for those requirements and route through it to Launch's origin.

**`CONTENTSTACK_REGION` SDK quirk**

- `CONTENTSTACK_REGION` is **not an officially recognised Contentstack SDK environment variable**; it was being used in application code to inject the region string at SDK initialisation.
- The SDK **defaults to the US region** when no region is provided, but if application code passes an **empty string** (e.g. because the env var is unset), the SDK constructs an invalid CDN hostname instead of falling back to the default. This causes `getaddrinfo ENOTFOUND` errors at build/request time.
- **Fix:** set `CONTENTSTACK_REGION` to `'us'` (not `'NA'`). The SDK region identifier for North America is **`us`**—using `NA` produces an incorrect hostname (`na-cdn.contentstack.com`). The documented values for non-US regions are `eu` and `azure-na`; the US region identifier is `us`.

**Suggested checks if you see a Contentstack CDN hostname error on Launch**

1. Confirm the **region identifier** passed to the SDK is `'us'`, `'eu'`, or the correct region string—not `'NA'` or another alias.
2. Ensure the env var is **set on the Launch project**, not just locally—Launch build machines do not inherit local environment.
3. If the region env var is intentionally omitted, verify the SDK initialisation code does not pass `undefined` or `''` as the region argument; omit the argument entirely to trigger the SDK default.

**Open / unresolved**

- After correcting the region env var, the deployed site returned **`CF001` Internal Server Error**. Root cause was not identified in this thread. See the CF001 troubleshooting page and check application-level errors in the Launch build/function logs.
