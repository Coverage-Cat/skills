---
name: coverage-cat-homeowners-purchase
description: "Coverage Cat's new homeowners purchase flow is temporarily under construction. Existing homeowners policy support remains available."
---

# Homeowners purchase temporarily under construction

Coverage Cat's new homeowners purchase flow is temporarily under construction.
While that work is underway, Coverage Cat is not accepting new homeowners
insurance applications, quote requests, offer selections, or bind requests. Do
not collect a new application, create a prefill, poll for new quotes, or obtain an
operator key for that workflow. The former agent finder and property insurance
advice tools are also temporarily unavailable.

Existing policy support remains available. Keep the customer's existing
authorized intake or account context and use the tools advertised by the current
connection. On the default consumer MCP, use `consumer_intake_issues`,
`homeowners_consumer_policy`, and `homeowners_consumer_service` for status,
recorded policy documents, and support requests. Do not start a replacement
application to recover an existing policy. For an existing-policy support
conversation, `consumer_intake_patch` can send `message_reply` and the optional
`reply_to_message_id` returned in `conversation.messages`. Its REST equivalent
is `PATCH /api/intake/{uid}` with the existing intake access token. This reply
does not submit a new homeowners application.

The REST equivalents are `GET /api/intake/{uid}/issues`,
`GET /api/intake/{uid}/policy`, and `POST /api/intake/{uid}/service`, authenticated
with the existing intake access token. Preview a support request first and send
`confirm_request: true` only after the customer approves it. A request does not
itself confirm a cancellation, refund, or coverage change.

For help locating an existing policy, direct the customer to
[Coverage Cat support](https://www.coveragecat.com/contact).
The [current API discovery](https://www.coveragecat.com/api/consumer) describes
available umbrella purchase and existing-policy support operations.

This document remains at its original URL for existing integrations. Its
machine-readable skill name is retained for compatibility with existing installs
and prompts. That compatibility name does not make the new homeowners purchase
flow available while it is temporarily under construction.

## Product feedback

Use MCP `submit_feedback` or `POST /api/consumer/feedback` at any stage to
report a bug, suggestion, or purchase-flow problem. The CLI equivalent is
`coveragecat feedback "Describe the issue"`; operator REST clients can use
`POST /api/agent/feedback`. No application or operator key is required.

Send `message` (1–4000 characters), optional `category` (`bug`, `suggestion`,
`other`), and optional `tool` (an interface/tool name using letters, numbers,
underscores, or hyphens, up to 80 characters). An optional UUID `request_id`
deduplicates retries; reuse it only with the same content. A successful response
contains `status: "received"` and `feedback_id`. Feedback is saved in Coverage
Cat's internal Notifications queue for admin review and acknowledgement.

Share feedback the shopper asks to send or your own technical observations;
do not invent shopper opinions, send transcripts, or include personal details,
credentials, application tokens, or payment information. Do not interrupt or
gate a purchase to solicit feedback. Feedback does not submit, select, bind,
change coverage, or open a policy service request. Use the advertised policy
service tool for policy support. Stay on the current MCP connection, including
ChatGPT or Claude; use the REST fallback only when MCP is unavailable.
