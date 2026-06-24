QUESTION

Customer **PODS** (Org `bltf391054f55c32754`, Azure NA, SF case **00057459**) wants to:
1. **Set up 2 new domains/environments:** `training.pods.com` and `training.pods.ca`.
2. **Decommission** the `pods-mock.azcontentstackapps.com` environment — delete it, or repurpose it for the training setup? What's the best route?

_Keywords: add new domains, set up environment, domain limit increase, decommission environment, delete environment, repurpose environment, subdomain training, multiple custom domains one environment, separate environment per domain._

ANSWER

## Adding the 2 new domains

- Adding domains requires the **domain entitlement to allow them** — the **domain limit was increased by 2** (existing custom apex domains left as-is) so the customer can add the two **`training.*` subdomains**. Have them add the subdomains and confirm success.
- Most of this is **self-serve** once the limit is in place.

## Decommissioning `pods-mock...` environment

- You can **safely shut down or delete an environment** if it's no longer in use, **or repurpose** that same environment for the training setup if that fits your workflow better. Both are fine — choose based on whether you want a clean new env or to reuse the existing one.

## Architecture choice — clarify before setup

Two supported patterns for the two training domains:
1. **Two separate environments**, each mapped to **one domain**, or
2. **A single environment** with **multiple custom domains** configured on it.

Both work; **setup steps differ** depending on the choice. Pick based on whether the two training sites are independent (separate envs) or the same app served on two domains (one env, multiple domains).

- Docs: Custom Domains (`/launch/custom-domain`), Environments overview (`/set-up-environments/about-environments`).

## Generalized guidance (for future similar queries)

- **"Add N new domains"** → confirm/raise the **domain limit** first (entitlement), keep existing apex domains intact, then the customer self-serves adding the (sub)domains.
- **"Decommission / delete an environment"** → safe to **delete** if unused, or **repurpose** it for the new use case — both supported.
- **"One environment with multiple domains vs one environment per domain?"** → both supported; choose per whether the domains serve the **same app** (one env, multiple custom domains) or **independent sites** (separate envs). Setup differs.

## Status

- Domain limit raised by 2; advised on delete-vs-repurpose for the old env and the env-per-domain vs multi-domain-per-env options. Awaiting customer to add subdomains / confirm.
