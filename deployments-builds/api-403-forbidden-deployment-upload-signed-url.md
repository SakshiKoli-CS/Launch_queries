QUESTION

**Johnson & Johnson Vision Care Inc** (SF **00056270**) reported a persistent `403 Forbidden` error (`launch.FORBIDDEN_RESOURCE`) when calling the Launch API to automate deployment uploads. The failing endpoint was:

```
GET {baseUrl}/projects/{projectUid}/environments/{envUid}/deployments/upload/signed_url
```

Both required headers — `authtoken` and `organization_uid` — were confirmed present and correct. The following endpoints worked without issue using the same credentials:

- `{baseUrl}/projects/upload/signed_url`
- `{baseUrl}/projects/{projectUid}`
- `{baseUrl}/projects/{projectUid}/environments/{envUid}`

The error occurred specifically on the **deployment upload signed URL** endpoint, not on read or project-level endpoints.

ANSWER

**Root cause — platform bug (JIRA CL-2963)**

The 403 on `deployments/upload/signed_url` was a **platform-side bug**, not a misconfiguration. The customer's auth headers were correct; the issue was in how the endpoint validated permissions for that specific operation.

This was not a user error. The fix to check first when this error appears: confirm that `authtoken` and `organization_uid` headers are both present. If they are and the error persists on this endpoint specifically, it is likely a platform regression.

**Suggested check for similar reports**

If `authtoken` + `organization_uid` are both present and other Launch API endpoints work but `deployments/upload/signed_url` returns `launch.FORBIDDEN_RESOURCE`:

1. Confirm the correct regional base URL is being used (e.g. `eu-launch-api.contentstack.com` for EU)
2. Confirm `organization_uid` is the org UID, not the stack API key
3. If headers are correct and other endpoints work → platform bug; escalate and reference CL-2963 as a prior instance

**Resolution**

Fix deployed to production (JIRA **CL-2963**). Customer confirmed the API call worked after the fix.
