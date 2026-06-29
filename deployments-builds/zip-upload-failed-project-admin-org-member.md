QUESTION

**MicroStrategy** (org `blt45a2a52367b47001`, US AWS, SF **00056508**) reported that one specific user (`rpillutla@microstrategy.com`) could not upload a zip file for an existing Launch deployment and received:

> *"File upload failed. This could be due to a network or a validation error. Please try again."*

The same zip file uploaded successfully for other users. The affected user had **Admin access at the project level** but only **Member access at the org level**. Permissions had not changed recently and uploads had worked for the same user a few days earlier. Troubleshooting — multiple browsers, incognito, re-login, downloading and re-uploading the zip — did not resolve the issue.

ANSWER

**Root cause — permission mismatch: project Admin + org Member (platform bug, CL-3298)**

The upload failure was **specific to the combination** of **Admin at the project level** and **Member at the org level**. This access-level mismatch caused the upload to fail silently with a generic network/validation error, even though the user had the expected project-level permissions.

This was confirmed by reproducing the issue internally with the same role combination. It is not related to the zip file contents, browser, network, or any recent change on the customer side.

**Fix**

Platform-side fix deployed (JIRA **CL-3298**). After the fix, the user was asked to retry the upload.

**Diagnostic pattern for similar reports**

If a zip upload fails for one user but works for others with the same file:

1. Check the affected user's **org-level role** — are they a Member (not Admin/Owner) at the org level?
2. Check their **project-level role** — are they an Admin at the project level?
3. If yes to both → this is the known CL-3298 pattern. Confirm the fix is deployed; if the issue persists, escalate as a regression.

The generic "network or validation error" message is misleading in this case — the actual cause is an access-handling bug, not a network or file issue.
