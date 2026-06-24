QUESTION

Customer **Conforama** (SF case **00057499**, GCP Europe) **cannot add new environment variables** — they've hit the per-environment limit in their integration environment. Docs state **100 environment variables per environment**, and they're at/over it. Can the limit be **increased**, or is there a **workaround**?

- Project `687e01f1e89dfe6aa0015a15`, Environment `687e07ab6a83ceb409ed92b5`, Org `blt1886e09a169ae0d8`.

_Keywords: environment variable limit, 100 env vars per environment, hit env var cap, can't add environment variable, increase env var limit, consolidate env vars, JSON env var, delimited env var, env config limit._

ANSWER

## The limit — 100 env vars per environment (hard, can't be raised)

- Environment variables are capped at **100 per environment**. This is a **hard limit** and **cannot be increased** at this time.

## Workarounds — reduce / consolidate

1. **Audit** the current variables — remove or consolidate any that are unused or redundant.
2. **Combine related values into a single variable** (delimited string or JSON) and **decode in the app at runtime**:

   **Delimited:**
   ```
   API_URLS=https://api1.com,https://api2.com,https://api3.com
   ```
   ```js
   const urls = process.env.API_URLS.split(",");
   ```

   **JSON:**
   ```
   APP_CONFIG={"apiUrls":["https://api1.com","https://api2.com","https://api3.com"]}
   ```
   ```js
   const config = JSON.parse(process.env.APP_CONFIG);
   ```

This keeps you within the 100-variable cap while still managing many configuration values.

## Generalized guidance (for future similar queries)

- **"Can't add more environment variables / hit the limit"** → **100 per environment** is a **hard cap** (not increasable currently).
- **Workaround:** audit & remove unused vars; **group related values into one var** via **delimited string or JSON**, decode at runtime — many logical settings, one variable slot.

## Status

- Confirmed **100/env hard limit** (no increase available); provided the **consolidation (delimited/JSON)** workaround. Customer acknowledged.
