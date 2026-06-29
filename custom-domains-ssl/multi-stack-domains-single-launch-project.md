QUESTION

A customer asked whether 5 additional domains could be split flexibly across multiple Contentstack stacks, or whether there is a strict one-to-one mapping between a stack and a Launch project. Specifically: can 2 domains be assigned to one stack and 3 domains to a second stack, all within a single Launch project?

The use case: one brand's content lives in Stack A, two other brands' content in Stack B — but all served from one Launch project.

ANSWER

**Domains are assigned to environments, not to stacks.**

Launch does not enforce any mapping between a Launch project and a Contentstack stack. Custom domains are assigned to **environments** within a Launch project. Each environment hosts an application, and that application's code is free to fetch content from any CMS stack, external API, database, or other source — Launch does not restrict this.

This means the customer's intended setup is fully supported:

- Environment 1: assigned 2 custom domains → application fetches from Stack A (Brand 1)
- Environment 2: assigned 3 custom domains → application fetches from Stack B (Brands 2 & 3)
- Both environments live within the same Launch project

There is no documentation for this specific multi-stack pattern because it is not a Launch configuration — it is simply how the customer writes their application code. Launch hosts whatever the application does.

**Key clarification**

The 5-domain add-on increases the **domain limit for the Launch project**. Those domains are then allocated to individual environments within that project. The split across environments (2 + 3, or any other combination) is entirely at the customer's discretion.

See also: [Add new domains + decommission/repurpose an environment](launch-add-new-domains-decommission-repurpose-environment.md) for domain-limit increase and environment setup details.
