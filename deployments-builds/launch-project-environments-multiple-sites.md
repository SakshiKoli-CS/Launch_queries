QUESTION

A partner (Huge) asked what **"Launch Projects: 1"** on an order form means, and whether there is a restriction on the number of websites that can be spun up within a single Launch project. They wanted to confirm that ≥ 3 websites are possible.

ANSWER

**What a Launch project encompasses**

A **Launch project** is the top-level unit within a Launch organization. It contains one or more **environments**, and a single GitHub repository connection.

| Concept | Scope |
|---|---|
| GitHub repo | Set at the **project** level — all environments in the project share the same repo (different branches are fine, but not different repos) |
| Environments | Multiple per project — can represent stages of one site (dev/staging/UAT/prod) **or** entirely different websites |
| Domains | 1 apex domain included by default; additional apex domains purchasable as add-ons |

**Multiple websites under one project**

There is **no hard restriction on the number of websites** within a single Launch project. Options:

- **Same codebase, multiple domains** → attach multiple domains to the same environment
- **Different environments per website** → each environment hosts a distinct site (subject to environment add-on limits)

Example with 3 sites on one project:
- `www.domain.com` (prod environment — Site A)
- `test.domain.com` (prod environment — Site B, separate environment)
- `abc.domain.com` (prod environment — Site C, separate environment)

By default, only 1 apex domain is included; the other domains would be subdomains unless additional apex domains are purchased.

**When a second project is required**

If each website uses a **different GitHub repository**, separate Launch projects are needed — the repo connection cannot differ between environments within the same project.

**Environment add-ons**

If more environments are needed than included in the plan (e.g. 3+ distinct websites each needing their own environment), additional environment add-ons may need to be purchased.

**Summary for order-form clarification**

"Launch Projects: 1" = one project in the Launch org. Within that project, ≥ 3 websites are possible as long as they share the same GitHub repo. If repos differ, one project per repo is needed.
