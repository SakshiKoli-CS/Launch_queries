QUESTION

Customer **Concord** (SF case **00058876**) reported that **Cloudflare Email Protection** appears enabled on their Launch environments, breaking **`mailto:` links** on pages that need visible email addresses. Their client's corporate **CSP policy enforces nonces**, which **blocks the Cloudflare-injected decoding script** from executing — so obfuscated email addresses are **never decoded** for end users and email links don't work.

They asked whether **Cloudflare Email Protection can be disabled** for their Launch project to restore functionality.

- **Org ID:** `blt4781beb962759a43`
- **Launch project:** `67b744cb0424db6a7fd21eba`
- Example pages: `https://www.twgjapan.com/お問い合わせ`

_Keywords: Cloudflare Email Protection, email obfuscation, mailto not working, CSP nonce blocks injected script, email decode script blocked, disable email protection per project, email_off, zone-level setting, obfuscated email addresses._

ANSWER

## Can Cloudflare Email Protection be disabled per project? → No

- **Cloudflare Email Protection cannot be disabled for an individual Launch project/environment.** It's configured at the **zone level** and is **shared across all customers** on the platform — so there's no per-account/per-project toggle.

## Why it breaks under a strict CSP

- Cloudflare Email Protection **obfuscates email addresses** and injects a **decoding script** to restore them client-side.
- A CSP that **enforces nonces** blocks that injected script (it has no matching nonce), so the email **never gets decoded** → broken/obfuscated `mailto:` links.

## Workaround — exclude specific emails from obfuscation

- Wrap the affected email addresses in Cloudflare's **`email_off` HTML comments** so Cloudflare **doesn't obfuscate** them (and therefore injects no decoding script → no CSP nonce conflict):
  ```html
  <!--email_off-->contact@example.com<!--/email_off-->
  ```
- Ref: Cloudflare docs — "Prevent Cloudflare from obfuscating email." This restores expected `mailto:` behavior on the affected pages.

## Customer perspective / longer-term

- Customer accepted that it can't be disabled per-account but noted they wouldn't expect this behavior from a hosting offering, and were reluctant to broadly wrap `mailto:` links / add HTML comments across pages (may use temporarily).
- **Planned long-term fix on their side:** transition from email links to **contact forms** (avoids email obfuscation entirely). Ticket closed.

## Generalized guidance (for future similar queries)

- **"Disable Cloudflare Email Protection / email obfuscation for my Launch project"** → **Not possible per project/environment** — it's a **zone-level, platform-shared** setting.
- **"`mailto:` / emails not decoding, broken under our CSP"** → the **CSP nonce blocks Cloudflare's injected decode script**. Exclude the specific emails with **`<!--email_off-->...<!--/email_off-->`** so they're not obfuscated.
- **Long-term:** **contact forms** instead of exposed email links sidestep the obfuscation/CSP conflict altogether.
