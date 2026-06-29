QUESTION

A customer (Lee Ablett) reported that the **bulk edit field** in the Launch UI felt too small at a default height of 3 lines and requested a larger input area.

ANSWER

**Default behavior (before fix)**

The bulk edit text area was fixed at a height of 3 lines. It was already resizable by dragging the bottom-right corner, but the low default made it feel restrictive.

**Fix — dynamic expansion (live on prod)**

The text area now expands dynamically based on input:

| Input size | Behaviour |
|---|---|
| No input or ≤ 3 lines | Box height stays at 3 lines |
| 4–6 lines | Box auto-expands to fit content |
| > 6 lines | Box stays at 6-line height; content scrolls |

**For similar reports**

If a customer says the bulk edit field is too small — the field now auto-expands up to 6 lines and scrolls beyond that. If they are on a version where this is not yet live, confirm the fix is deployed to prod.
