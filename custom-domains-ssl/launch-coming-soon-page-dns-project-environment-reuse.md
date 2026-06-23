QUESTION

Customer **Cherokee Nation Entertainment (CNE)** wants to show a **"Coming Soon" page** for a new site (`statusplusrewards.com`) before the full site is ready. Two questions:

1. **DNS entries:** Can they work with the Launch team to **get the DNS entries to add** so the Coming Soon site is publicly accessible?
2. **Project quota / reuse:** They've used ~3–4 Launch projects and want to keep the **4th project for a new casino property ("Gold Strike")**. Does the Coming Soon page need its **own Launch project**? Can they use a project **temporarily** for the Coming Soon page and then **repurpose it** for Gold Strike?

_Keywords: coming soon page, pre-launch placeholder, custom domain DNS records, who provides DNS entries, Launch does not provide DNS manually, project quota, project slot, reuse/repurpose project, new environment, temporary environment, custom domain per environment._

ANSWER

## 1. DNS entries for the Coming Soon page

- This is achieved using **Launch's Custom Domain** capability.
- **The Launch team does NOT manually provide DNS records.** When a custom domain is configured in the **Launch UI**, the required DNS records are **automatically generated and displayed** during setup.
- The customer then **adds those records at their DNS provider** to complete domain verification and make the site publicly accessible.
- Ref: Custom Domain documentation.

## 2. Project quota & reusing a project for Gold Strike

**Preferred — avoid consuming an extra project slot:**
- If the customer **already has a Launch project planned** for the final `statusplusrewards.com` site, create a **new environment within that project** for the temporary Coming Soon page and **configure the custom domain on that environment**. This avoids using an additional project slot and **preserves the remaining project for Gold Strike**.

**Alternative — if no project is planned for statusplusrewards.com yet:**
- They can use their available **4th project slot** for the Coming Soon page now, then **repurpose that same project** for Gold Strike later by **deploying the Gold Strike application and updating the custom domain configuration**.
- During that transition, any **temporary environments and custom-domain mappings** for the Coming Soon page must be **updated or removed**.
- Ref: New Environment Creation documentation.

## Ownership note

- This is a standard Launch how-to (custom domain + project/environment structuring) — handled via the normal **support channel**; no special Launch-team ownership required.

## Generalized guidance (for future similar queries)

- **"Can the Launch team give us the DNS records?"** → No manual DNS provisioning. **Create the custom domain in the Launch UI**, then the **auto-generated records are shown** for the customer to add at their DNS provider.
- **"Do we need a separate project for a Coming Soon / temporary page?"** → Prefer a **new environment in an existing/planned project** (with its own custom domain) to **avoid burning a project slot**.
- **"Can we reuse/repurpose a project later?"** → Yes — redeploy the new app and **update the custom-domain config**; clean up temporary environments and stale domain mappings during the switch.
- **Custom domains are configured per environment**, so a temporary placeholder and the eventual site can live as separate environments under one project.
