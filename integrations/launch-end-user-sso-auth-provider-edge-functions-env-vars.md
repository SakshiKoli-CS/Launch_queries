QUESTION

Does **Launch support Single Sign-On (SSO) / end-user authentication** for websites hosted on it (i.e. real visitors of a live Launch site logging in)? Customer (**Sky UK**, evaluating Launch for a new Brand Portal) wants confirmation of this understanding before adopting:

> Launch supports end-user auth **via code** with **any provider** (Auth0, Clerk, Firebase, AWS Cognito), using **Environment Variables** to store client secrets/IDs and **Edge / Serverless Functions** to handle the auth handshake/callback server-side.

_Keywords: SSO end-user authentication Launch, login for website visitors, Auth0 Clerk Firebase Cognito, framework-agnostic auth, no built-in SSO no identity management, environment variables client secret, edge functions OAuth callback token exchange, edge SSO example._

ANSWER

## Yes — Launch supports end-user auth via code (BYO provider); it doesn't manage identities itself

- **Launch does not manage end-user identities or provide built-in SSO.** Instead it's **framework-agnostic** and lets you implement end-user authentication **in code with any provider** (Auth0, Clerk, Firebase, AWS Cognito, etc.).
- Launch provides the **platform capabilities** to do this securely:
  - **Environment Variables** — to **store client secrets/IDs securely.**
  - **Edge Functions / Serverless Functions** — to **handle the auth handshake/callback** (OAuth callbacks, token exchanges) **server-side.**

## Recommended approach — handle auth at the Edge

- **Use Edge Functions** for OAuth callbacks and token exchanges. Benefits: **low latency**, keeps **sensitive logic server-side**, and stays **framework-agnostic**, fully managed within the application.
- Many customers have successfully implemented **SSO at the edge** this way.
- References: **Launch Edge-SSO example** + **Edge Functions documentation**.

## Generalized guidance (for future similar queries)

- **"Does Launch support SSO / end-user login?"** → **Yes, via code with any auth provider** — Launch **doesn't provide built-in SSO or manage identities**, but supplies the building blocks: **env vars** (secrets) + **Edge/Serverless Functions** (auth flow).
- **Recommended pattern:** run the **OAuth callback / token exchange in an Edge Function** (low latency, server-side secrets, framework-agnostic) — see the **Launch Edge-SSO example**.
- Distinguish **end-user/website-visitor auth** (this entry — app-implemented) from **Contentstack/Launch *console* SSO** (org-admin login to the product), which is a separate concern.

## Status

- **Confirmed accurate.** Launch supports **end-user SSO/auth via code with any provider**, using **env vars** + **Edge Functions** (recommended for callbacks/token exchange); no built-in SSO/identity management. Response approved to share with the customer (Sky UK).
