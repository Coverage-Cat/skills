---
name: coverage-cat-homeowners-purchase
description: "Retirement guidance for Coverage Cat homeowners purchase integrations and support for existing homeowners policies. New homeowners applications and quotes are unavailable."
---

# Homeowners applications retired

Coverage Cat no longer accepts new homeowners insurance applications, quote
requests, offer selections, or bind requests. Do not collect a new application,
create a prefill, poll for new quotes, or obtain an operator key for that workflow.
The former agent finder and property insurance advice tools are also retired.

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

This document remains at its original URL for existing integrations. Its legacy
skill name is retained for compatibility; it no longer starts homeowners purchases.
