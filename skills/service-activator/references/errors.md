# Error handling

Every tool returns `statusCode`. 200 means success; anything else carries `errorDescription`.

## What to show the user

When a tool returns an error, show the user **only the error message**, nothing else:

- No status code (400, 401, 500), no error code (`1027`, `263`, `929`, …), no IDs, no field names, no raw response body.
- Server errors that arrive as `400-<code>-<message>` (e.g. `400-263-Customer can not be changed`): show only the `<message>` part → "Customer can not be changed".
- Show the message as it is: do not add explanations, causes, suggested fixes or tables around it.
- 401 / Unauthorized: show only "Unauthorized".
- A lookup tool fails without a message: show only "Something went wrong".

This overrides any error explanation in the flow above: flow-specific error tables are for you to understand the error, not to show to the user.

## What to do after an error

- **400:** after showing the message, wait for the user. Call again only after the user answers with a change.
- **401:** stop; do not call any more tools.
- **500:** never retry automatically, because the record may have been created despite the error. Do not create it again unless the user confirms it was not created.
- Never call a create tool twice for the same request, and never retry a failed create silently.
