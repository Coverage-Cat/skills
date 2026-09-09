---
name: homeowners-insurance-compare-and-purchase
description: "Discoverability alias for Coverage Cat's homeowners purchase skill. Use whether you're a consumer's own AI agent or a partner operator: start with the consumer-prefill handoff when no operator bearer key is available, or use the delegated homeowners flow when one is."
---

# Homeowners Insurance Compare And Purchase

This alias mirrors Coverage Cat's canonical `coverage-cat-homeowners-purchase` skill.

Use it when a shopper wants their own AI agent to gather context before a Coverage Cat handoff, or when a partner-operator agent needs Coverage Cat to run the delegated homeowners quote loop.

## Start Here

1. If your runtime supports MCP, default to Coverage Cat's no-OAuth consumer MCP. Register `/.well-known/mcp.json` in ChatGPT and other manifest-aware hosts, or configure `https://www.coveragecat.com/api/consumer/mcp` as the direct transport URL for local Codex. Default consumer-operated homeowners runs to the consumer MCP subset: `homeowners_consumer_prefill`, `consumer_intake_issues`, and `consumer_intake_patch`. If Coverage Cat MCP tools are unavailable in a runtime that supports MCP, stop and ask a short choice question instead of telling the user to register `/.well-known/mcp.json` immediately.

   Ask the question on its own line exactly as:

   Would you like to install the Coverage Cat MCP (recommended) or use the REST/CLI tools instead?

   Only share `/.well-known/mcp.json` after the user chooses MCP or asks for setup details.
2. Fetch `GET /api/agent`.
3. Fetch `GET /api/agent/openapi.yaml` or `GET /openapi.json`.
4. Read `GET /api/agent/homeowners/skill.md`.
5. Read `/ai/skills/homeowners/setup` if you need the delegated operator path.

## Choose The Path

1. Use the consumer-prefill path when there is no operator key or the shopper wants their own AI agent to assemble the application from user-controlled context before the handoff.
2. On the default consumer MCP, stay on that same consumer-operated subset through review and quote follow-up. Final homeowners bind still happens in Coverage Cat's consumer portal rather than the delegated MCP tools. `GET https://www.coveragecat.com/api/consumer/mcp` returning `405 Method Not Allowed` is expected because the direct transport uses `POST` JSON-RPC, and `/.well-known/mcp.json` is discovery metadata rather than the transport endpoint.
3. Use the delegated operator MCP or delegated homeowners API only when you already have a real operator bearer key or an OAuth-capable host that can complete delegated auth.
3. Use the delegated operator path only when you have a real Coverage Cat operator bearer key and approved back-office context to prefill the application.
4. If your runtime cannot prefill, fall back to the direct browser handoff at `/intake`.
5. Do not mix the consumer-prefill and delegated paths in one session.

## Delegated Homeowners Loop

- `POST /api/agent/homeowners/quotes`
- `GET /api/agent/homeowners/dashboard`
- `POST /api/agent/homeowners/dashboard/session`
- `POST /api/agent/homeowners/fix-issues-email`

## Guardrails

- On a cold start, ask only for the user's full name, email, and property address. Then search the user's own context, property records, Zillow, and Realtor.com before asking anything else.
- Before you call Coverage Cat, recover core shopper and occupancy facts your runtime can defensibly find from user-controlled context, especially date of birth, marital status, and whether the home is owner-occupied or a new purchase.
- Do not open with a date-of-birth or marital-status questionnaire when the consumer-prefill handoff can already carry an estimated review card.
- Do not break the second turn into a standalone current-policy-expiration question. Keep current-policy expiration in the review turn as an estimated value two months from today when needed.
- On every pre-submit user-facing turn, say explicitly that the application is not submitted yet and Coverage Cat has not received a submitted application yet.
- When you list gathered details, estimated answers, or remaining items for the homeowner, use short labeled bullets or sections rather than a prose paragraph.
- When any shown value is estimated, mark that bullet or value with `*`, include the short note `* = estimated` once above and once below the list, and do not prefix every estimated line with `[Estimated]`.
- For dropdown-like homeowners fields such as home ownership / occupancy and property type, numbered options are fine and the number alone as shorthand is acceptable.
- Keep the review output split into clearly labeled applicant details, property details, estimated structure details, estimated systems details, and other sections so the review does not blur together.
- Use `POST /api/consumer/homeowners/prefill` plus one review card and soft-credit consent when you do not have an operator bearer key.
- When the default consumer MCP is available, use the consumer-operated subset named above instead of switching to delegated homeowners tools without operator auth.
- Reuse the same `uid` for the delegated session.
- Use `sandbox: true` only on the first delegated create call when rehearsing.
- Hold back `credit_check_authorized` until the real homeowner reviews the assembled application and explicitly says yes to the soft-credit prompt.
- Expect `202 Accepted` / `pending_quotes` semantics for long-running delegated quote work and keep polling asynchronous.
