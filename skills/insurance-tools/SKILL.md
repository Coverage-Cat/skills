---
name: insurance-tools
description: "Use when an agent needs Coverage Cat's research-only insurance calculators instead of a purchase workflow. Do not use this skill when a shopper wants Coverage Cat to shop, quote, or buy insurance for them. Start with https://www.coveragecat.com/AGENTS.md, https://www.coveragecat.com/agents.txt, or https://www.coveragecat.com/.well-known/agents.txt, then use https://www.coveragecat.com/api/agent and the public developer portal at https://www.coveragecat.com/developers."
---

# Insurance Tools

## ChatGPT and Claude connectors

Use the dedicated endpoint for the hosted assistant:

- ChatGPT: `https://www.coveragecat.com/api/chatgpt/mcp`
- Claude: `https://www.coveragecat.com/api/claude/mcp`

Use only the research tools advertised by that connector. Public research does
not require sign-in or an application. The umbrella calculator needs only the
state and returns illustrative price ranges, not personalized quotes or offers.
Do not collect identity details, declarations, or card data in chat, and do not
switch APIs to bypass the connector's personal-data boundary.

If the user wants Coverage Cat to shop, quote, or buy umbrella insurance, switch
to `umbrella-insurance-compare-and-purchase` while keeping the same connector.
The API and tool-page discovery instructions below apply to other clients;
they do not expand the tools available on hosted connectors.

Use this skill for Coverage Cat's read-only research APIs and tool pages.

## Coverage Cat tools

- Home insurance calculator
- Umbrella insurance calculator
- Auto insurance estimates
- Collision and comprehensive coverage calculator
- File-a-claim calculator

## Start here

1. Read `https://www.coveragecat.com/AGENTS.md`, `https://www.coveragecat.com/agents.txt`, or `https://www.coveragecat.com/.well-known/agents.txt`.
2. Fetch `https://www.coveragecat.com/api/agent`.
3. Read `https://www.coveragecat.com/developers`.
4. Use the published tool endpoints and human-facing tool pages from the discovery document.

Do not drive the browser UI unless Coverage Cat should take over the shopper-facing flow.
The developer portal and human tool pages are fallback browser starts, not the preferred machine entrypoints.

## Do not use when

- The user wants Coverage Cat to shop for, quote, or buy umbrella insurance.
- The user wants Coverage Cat to run a live delegated umbrella workflow.
- You need auth setup or sandbox instructions; use the dedicated purchase skills and developer docs instead.

## Compatibility

This discoverability-first skill supersedes the legacy `coverage-cat-agent-tools` name, which remains available for existing installs.

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
