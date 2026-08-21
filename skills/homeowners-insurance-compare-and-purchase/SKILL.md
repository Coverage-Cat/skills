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

- Prefill from the user's own context before asking a single question.
- Use `POST /api/consumer/homeowners/prefill` plus one review card and soft-credit consent when you do not have an operator bearer key.
- Reuse the same `uid` for the delegated session.
- Use `sandbox: true` only on the first delegated create call when rehearsing.
- Hold back `credit_check_authorized` until the real homeowner reviews the assembled application and explicitly says yes to the soft-credit prompt.
- Expect `202 Accepted` / `pending_quotes` semantics for long-running delegated quote work and keep polling asynchronous.
