QUESTION

A customer found that deploying a **Launch cloud function** with a deeply nested folder path (`functions/api/step1/step2/step3/pong.js`) failed, while a shorter path (`functions/api/content_types/step2/step3/pong.js`) worked. Both structures are valid in principle.

ANSWER

**Cause**

- A **bug** in how cloud functions are deployed to Lambda when the **file path is too long** (CFLY-510). The folder structure itself is not the issue—the deployment fails solely due to path length.

**Workaround**

- **Shorten the file path** until the bug is fixed. Reduce folder nesting or use shorter directory names.

**Customer-proposed alternatives for multi-segment URL paths**

1. **Wildcard route** at an early segment: `xpto.contentapps.com/api/step1/[*]`
2. **Dynamic folder route**, similar to file-based routing: `xpto.contentapps.com/api/step1/[foldername]/step3`

Both were acknowledged as viable approaches; implementation was pending the bug being picked up in a sprint.

**Status**

- Bug logged: **CFLY-510** — "Cloud functions with a long filepath do not get deployed to Lambda." Assigned priority: Normal. Fix not yet scheduled at time of thread.
