---
name: umbrella-insurance-compare-and-purchase
description: "Discoverability alias for Coverage Cat's umbrella purchase skill. Use whether you're a consumer's own AI agent or a partner operator: start with the consumer-prefill handoff when no operator bearer key is available, keep the direct `/api/intake/:uid/...` follow-up loop in chat when possible, or use the delegated umbrella flow when a real operator key is already present."
---

# Umbrella Insurance Compare And Purchase

This alias mirrors Coverage Cat's canonical `coverage-cat-umbrella-purchase` skill.

Use it when a shopper wants their own AI agent to gather context before a Coverage Cat handoff, or when a partner-operator agent needs Coverage Cat to run a live umbrella quote workflow instead of just returning read-only estimates.

## Start Here

1. If your runtime supports MCP, register `/.well-known/mcp.json` and default consumer-operated umbrella runs to the product MCP subset: `umbrella_consumer_prefill`, `consumer_intake_issues`, `consumer_intake_patch`, `umbrella_consumer_select`, `umbrella_consumer_bind`, and `umbrella_consumer_attach`.
2. Fetch `GET /api/agent`.
3. Fetch `GET /api/agent/openapi.yaml` or `GET /openapi.json`.
4. Read `GET /api/agent/skill.md`.

## Choose The Path

1. Use the consumer-prefill path when there is no operator key or when the shopper wants their own AI agent to assemble the application from user-controlled context before the handoff.
2. On the product MCP, keep consumer-operated umbrella runs on that same subset through quote review, declarations upload, `payment_needed`, and `payment_url`; do not switch to delegated tools just to reach checkout.
3. Use the delegated operator path only when you already have a real Coverage Cat operator bearer key and approved back-office context.
4. Do not mix the consumer-prefill and delegated paths in one session.

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
- When the product MCP is available, use the consumer-operated subset named above instead of switching to delegated umbrella tools without operator auth.
- On every pre-submit user-facing turn, say explicitly that the application is not submitted yet and Coverage Cat has not received a submitted application yet.
- When you list gathered details, estimated answers, or remaining items for the shopper, use short labeled bullets or sections rather than a prose paragraph.
- When any shown value is estimated, mark that bullet or value with `*`, include the short note `* = estimated` once above and once below the list, and do not prefix every estimated line with `[Estimated]`.
- Keep the user out of the loop until `GET /api/intake/:uid/issues` reaches `ready_for_submission`, or until that payload already carries a staged `review` plus bundled `next_question` for the shopper-owned details Coverage Cat still needs.
- On that staged review turn, ask for the shopper-owned bundle together in plain English rather than one field at a time. Keep the assembled review and the requested items in labeled bullets or sections. When Coverage Cat asks for full name, email, phone, full address, birthday, marital status, driver's license state, driver's license number, and net worth, keep that as one turn and let the user send corrections in the same reply. If marital status means Coverage Cat also asks for spouse or partner full name, spouse date of birth, spouse driver's license state, or spouse driver's license number, keep that spouse bundle in the same message too instead of opening a second follow-up.
- If the shopper gives a birthday in normal US `MM-DD-YYYY` form, normalize it to `YYYY-MM-DD` before you patch Coverage Cat.
- Keep the returned `uid` plus the freshest rotated `intake_access_token`, continue the direct `/api/intake/:uid/...` follow-up loop in chat, and use `resume_url` only as a browser fallback.
- Use an operator-issued bearer token for every delegated umbrella endpoint.
- Start with the fullest intake and any matching `field_estimates`.
- If delegated `draft` reuses or conflicts on a recent buyer-email session, switch to the returned `uid` and continue that existing application instead of retrying create.
- Do not ask for user credit consent until the user has chosen an offer and Coverage Cat requests it at `select`.
- When Coverage Cat returns multiple offers, do not collapse them to only the recommended default if alternatives are present. Render offers as a single markdown table with columns `Carrier | Coverage limit | Annual price | Min. auto limits | Notes` (one row per offer, recommended first, `(Recommended)` in the Notes column of the recommended row). Format prices as `$941/yr` and coverage limits as `$2M` / `$3M`. Do not restate offer prices or coverage limits in the surrounding prose.
- Below the offers table, always render a `**Next steps to purchase:**` bulleted checklist. Include `Pick an offer`, `Confirm your auto policy meets the Min. auto limits shown` (when any offer has minimum_auto_limits), the Monoline/Markel declarations-upload bullets when applicable (about 5-10 minutes to checkout after upload), and the reminder to say `Yes` to the soft-credit-pull consent after selection. Never bury these steps in a prose paragraph.
- Keep payment and carrier-managed signing in the browser. When Coverage Cat returns `payment_needed`, share `payment_url` instead of collecting payment details in chat.
- For Monoline and Markel, keep required home and auto declarations in the post-choose flow. Once those uploads are on file, `documents_pending_review` usually advances to `payment_needed` with `payment_url` within about 5-10 minutes while declarations verification continues in parallel.
- If you are rehearsing, set `sandbox: true` only on the first create call for that `uid`.
- Keep read-only calculator and finder jobs on the separate insurance-tools skill rather than opening a delegated umbrella session.
