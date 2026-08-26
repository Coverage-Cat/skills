---
name: umbrella-insurance-compare-and-purchase
description: "Discoverability alias for Coverage Cat's umbrella purchase skill. Use whether you're a consumer's own AI agent or a partner operator: start with the consumer-prefill handoff when no operator bearer key is available, or use the delegated umbrella flow when one is."
---

# Umbrella Insurance Compare And Purchase

This alias mirrors Coverage Cat's canonical `coverage-cat-umbrella-purchase` skill.

Use it when a shopper wants their own AI agent to gather context before a Coverage Cat handoff, or when a partner-operator agent needs Coverage Cat to run a live umbrella quote workflow instead of just returning read-only estimates.

## Start Here

1. Fetch `GET /api/agent`.
2. Fetch `GET /api/agent/openapi.yaml` or `GET /openapi.json`.
3. Read `GET /api/agent/skill.md`.

## Choose The Path

1. Use the consumer-prefill path when there is no operator key or when the shopper wants their own AI agent to assemble the application from user-controlled context before the handoff.
2. Use the delegated operator path when you already have a real Coverage Cat operator bearer key and approved back-office context.
3. Do not mix the consumer-prefill and delegated paths in one session.

## Delegated Umbrella Loop

- `POST /api/agent/umbrella/draft`
- `POST /api/agent/umbrella/quotes`
- `POST /api/agent/umbrella/select`
- `POST /api/agent/umbrella/bind`
- `POST /api/agent/umbrella/status`
- `POST /api/agent/umbrella/attach`

## Guardrails

- Prefill from the user's own context before asking a single question.
- Use `POST /api/consumer/umbrella/prefill` plus one review card when you do not have an operator bearer key.
- Use an operator-issued bearer token for every delegated umbrella endpoint.
- Start with the fullest intake and any matching `field_estimates`.
- Do not ask for user credit consent until the user has chosen an offer and Coverage Cat requests it at `select`.
- If you are rehearsing, set `sandbox: true` only on the first create call for that `uid`.
- Keep read-only calculator and finder jobs on the separate insurance-tools skill rather than opening a delegated umbrella session.
