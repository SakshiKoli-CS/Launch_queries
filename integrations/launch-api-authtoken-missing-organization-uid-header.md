QUESTION

Customer (Org `blt8cfe6eeb3339532e`, SF case **00053959**) is calling the **Launch API with an auth token** and getting an **error** on `GET /projects`. Their request only sends the `authtoken` header:

```bash
curl --location 'https://azure-eu-launch-api.contentstack.com/projects' \
  --header 'authtoken: b***'
```

What are they missing?

_Keywords: Launch API auth token error, authtoken header, missing organization_uid, GET /projects error, organization header required, auth token based authentication, azure-eu-launch-api._

ANSWER

## Cause — missing the `organization_uid` header

- Auth-token-based requests to the Launch API require **both** the **`authtoken`** header **and** the **`organization_uid`** header.
- The request above sends **only `authtoken`** → the error is **expected**. Add the **`organization_uid`** header for the target org.
- Reference: **Auth Token-based Authentication** section of the Launch API docs — https://www.contentstack.com/docs/developers/apis/launch-api

## Generalized guidance (for future similar queries)

- **"Launch API call with an auth token errors (e.g. on `/projects`)"** → first **confirm `organization_uid` is being sent alongside `authtoken`**. Missing it is the common cause and the error is expected behavior, not a bug.
- This is distinct from **M2M / OAuth** auth (client credentials, `launch:manage` scope, region host) — for auth-token auth, the org header is the key requirement. See the [M2M CI/CD entry](launch-m2m-cicd-deployment-api-app-region-host.md) for the token-based alternative.
- Region host still matters: use the correct regional base URL (here `azure-eu-launch-api…`).

## Status

- **Resolved.** Cause was the **missing `organization_uid` header**; pointed to the Auth Token-based Authentication docs. Customer thanked the team.
