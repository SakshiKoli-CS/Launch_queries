QUESTION

Customer **McKinsey** (SF case **00045337**) reports their **preview environment** throws **"uses an unsupported protocol"** in the browser. The default Launch URL is:
```
https://demo.mckinsey.com.contentstackapps.com/
```
i.e. the chosen **project/subdomain name contains dots** (`demo.mckinsey.com`), producing a **multi-level subdomain** under `*.contentstackapps.com`.

_Keywords: uses an unsupported protocol error, cipher mismatch, dot in subdomain name, demo.mckinsey.com.contentstackapps.com, multi-level subdomain contentstackapps, wildcard certificate single label only, TLS handshake fails, rename project subdomain no dots use hyphens, preview environment error._

ANSWER

## Cause — a dot (`.`) in the subdomain breaks the wildcard TLS cert

- The default domain uses a **wildcard certificate** for **`*.contentstackapps.com`**. A wildcard **only matches ONE label** — it covers `foo.contentstackapps.com` but **NOT `foo.bar.contentstackapps.com`.**
- The chosen name **`demo.mckinsey.com`** contains **dots**, so the URL becomes **`demo.mckinsey.com.contentstackapps.com`** — a **multi-level subdomain** the wildcard cert **does not cover** → **TLS/cipher mismatch** → the browser reports **"uses an unsupported protocol."**

## Fix — don't use dots in the subdomain; use hyphens

- **Rename the project/subdomain to avoid dots** — replace them with **hyphens**, e.g. **`demo-mckinsey-com`** → `https://demo-mckinsey-com.contentstackapps.com/`, which is a **single label** the `*.contentstackapps.com` wildcard cert covers → valid TLS.

## Generalized guidance (for future similar queries)

- **"uses an unsupported protocol" / cipher mismatch on a `*.contentstackapps.com` default URL** → check the **subdomain for dots.** The **wildcard cert covers one label only**; a name with dots yields a **multi-level subdomain** the cert doesn't match → TLS fails.
- **Fix: use hyphens (not dots) in the Launch project/subdomain name** (`demo-mckinsey-com`, not `demo.mckinsey.com`).
- This is about the **default `contentstackapps.com` domain naming**, separate from adding a real **custom domain** (which gets its own cert).
- Related: [SSL untrusted on old browsers → missing cross-signed root](launch-ssl-untrusted-old-browsers-missing-cross-signed-root-sectigo-r46-add-to-chain-fib.md), [custom domain pending validation → DCV records](launch-custom-domain-pending-validation-dcv-records.md), [add SAN domains / managed cert / DCV](launch-add-san-domains-managed-cert-100-limit-format-renewal-cname-dcv-7day.md).

## Status

- **Resolved.** The **`.` in the subdomain** (`demo.mckinsey.com.contentstackapps.com`) made a **multi-level subdomain not covered by the `*.contentstackapps.com` wildcard cert** → **cipher mismatch / "unsupported protocol."** Fix: **rename to use hyphens** (e.g. `demo-mckinsey-com`) so it's a single label the wildcard cert covers.
