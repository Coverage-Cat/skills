---
name: homeowners-insurance-compare-and-purchase
description: "Discoverability alias for Coverage Cat's homeowners purchase skill. Use whether you're a consumer's own AI agent or a partner operator: start with the consumer-prefill handoff when no operator bearer key is available, or use the delegated homeowners flow when one is."
---

# Homeowners Insurance Compare And Purchase

This alias mirrors Coverage Cat's canonical `coverage-cat-homeowners-purchase` skill.

Use it when a shopper wants their own AI agent to gather context before a Coverage Cat handoff, or when a partner-operator agent needs Coverage Cat to run the delegated homeowners quote loop.

## Start Here

1. Fetch `GET /api/agent`.
2. Fetch `GET /api/agent/openapi.yaml` or `GET /openapi.json`.
3. Read `GET /api/agent/homeowners/skill.md`.
4. Read `/ai/skills/homeowners/setup` if you need the delegated operator path.

## Choose The Path

1. Use the consumer-prefill path when there is no operator key or the shopper wants their own AI agent to assemble the application from user-controlled context before the handoff.
2. Use the delegated operator path when you have a real Coverage Cat operator bearer key and approved back-office context to prefill the application.
3. If your runtime cannot prefill, fall back to the direct browser handoff at `/intake`.
4. Do not mix the consumer-prefill and delegated paths in one session.

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
- Reuse the same `uid` for the delegated session.
- Use `sandbox: true` only on the first delegated create call when rehearsing.
- Hold back `credit_check_authorized` until the real homeowner reviews the assembled application and explicitly says yes to the soft-credit prompt.
- Expect `202 Accepted` / `pending_quotes` semantics for long-running delegated quote work and keep polling asynchronous.
