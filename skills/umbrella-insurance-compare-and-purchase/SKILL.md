---
name: umbrella-insurance-compare-and-purchase
description: "Discoverability alias for Coverage Cat's umbrella purchase skill. Use whether you're a consumer's own AI agent or a partner operator: start with the consumer-prefill handoff when no operator bearer key is available, keep the direct `/api/intake/:uid/...` follow-up loop in chat when possible, or use the delegated umbrella flow when a real operator key is already present."
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

## Consumer Follow-Up Loop

- `POST /api/consumer/umbrella/prefill`
- `GET /api/intake/:uid/issues`
- `PATCH /api/intake/:uid`
- `POST /api/intake/:uid/select`
- `POST /api/intake/:uid/bind`
- `POST /api/intake/:uid/attach`

## Guardrails

- Prefill from the user's own context before asking a single question.
- On a cold start, call `POST /api/consumer/umbrella/prefill` before you ask the shopper a questionnaire. Send the fullest estimate you can justify from the shopper's own context first.
- On every pre-submit user-facing turn, say explicitly that the application is not submitted yet and Coverage Cat has not received a submitted application yet.
- When you list gathered details, estimated answers, or remaining items for the shopper, use short labeled bullets or sections rather than a prose paragraph.
- Keep the user out of the loop until `GET /api/intake/:uid/issues` reaches `ready_for_submission`, or until that payload already carries a staged `review` plus bundled `next_question` for the shopper-owned details Coverage Cat still needs.
- On that staged review turn, ask for the shopper-owned bundle together in plain English rather than one field at a time. Keep the assembled review and the requested items in labeled bullets or sections. When Coverage Cat asks for full name, email, phone, full address, birthday, marital status, driver's license state, driver's license number, and net worth, keep that as one turn and let the user send corrections in the same reply.
- If the shopper gives a birthday in normal US `MM-DD-YYYY` form, normalize it to `YYYY-MM-DD` before you patch Coverage Cat.
- Keep the returned `uid` plus the freshest rotated `intake_access_token`, continue the direct `/api/intake/:uid/...` follow-up loop in chat, and use `resume_url` only as a browser fallback.
- Use an operator-issued bearer token for every delegated umbrella endpoint.
- Start with the fullest intake and any matching `field_estimates`.
- Do not ask for user credit consent until the user has chosen an offer and Coverage Cat requests it at `select`.
- When Coverage Cat returns multiple offers, do not collapse them to only the recommended default if alternatives are present.
- Keep payment and carrier-managed signing in the browser. When Coverage Cat returns `payment_needed`, share `payment_url` instead of collecting payment details in chat.
- If you are rehearsing, set `sandbox: true` only on the first create call for that `uid`.
- Keep read-only calculator and finder jobs on the separate insurance-tools skill rather than opening a delegated umbrella session.
