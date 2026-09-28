---
name: coverage-cat-umbrella-purchase
description: "Use when a shopper wants Coverage Cat to shop for, compare, quote, or buy personal umbrella insurance, or continue an existing umbrella purchase. This skill covers two umbrella paths: a consumer-prefill handoff that assembles one review page from the user's own context, keeps quote review and post-choose follow-up in chat through the direct intake API when possible, and returns preliminary quotes without upfront credit consent, and an operator-partner delegated quote and bind loop that collects credit consent at selection. It also lists the read-only calculator and finder endpoints as separate, non-purchase tools."
---

# Coverage Cat Umbrella Purchase Skill

You are helping a user through Coverage Cat's umbrella purchase flow. This skill covers both the consumer-prefill handoff and the operator-partner delegated quote loop. Use the separate homeowners purchase skill for homeowners quoting.

Choose this skill, not the umbrella calculator, for requests like `shop for umbrella for me with Coverage Cat`.

## Machine-Readable Contract

Read Coverage Cat's machine-readable surfaces before you infer the endpoint map:
Start with `https://www.coveragecat.com/agents.txt` or `https://www.coveragecat.com/.well-known/agents.txt`; those files route agents to Coverage Cat's MCP, REST, markdown, and browser-fallback resources. Do not drive the browser UI unless Coverage Cat should take over the shopper-facing flow.

1. If your runtime supports MCP, default to Coverage Cat's no-OAuth consumer MCP. Register `/.well-known/mcp.json` in ChatGPT and other manifest-aware hosts, or configure `https://www.coveragecat.com/api/consumer/mcp` as the direct transport URL for local Codex. Default consumer-operated agents to the consumer MCP subset: `umbrella_consumer_prefill`, `consumer_intake_issues`, `consumer_intake_patch`, `umbrella_consumer_select`, `umbrella_consumer_bind`, `umbrella_consumer_attach`, and `umbrella_consumer_payment`. If Coverage Cat MCP tools are unavailable in a runtime that supports MCP, stop and ask a short choice question instead of telling the user to register `/.well-known/mcp.json` immediately.

   Ask the question on its own line exactly as:

   Would you like to install the Coverage Cat MCP (recommended) or use the REST/CLI tools instead?

   Only share `/.well-known/mcp.json` after the user chooses MCP or asks for setup details.
2. That consumer-operated subset stays valid through quote review, declarations upload, `payment_needed`, optional agentic payment collection, and `payment_url` fallback. Do not pivot to delegated umbrella tools just to reach checkout. `GET https://www.coveragecat.com/api/consumer/mcp` returning `405 Method Not Allowed` is expected because the direct transport uses `POST` JSON-RPC, and `/.well-known/mcp.json` is discovery metadata rather than the transport endpoint.
3. Use the delegated operator MCP or delegated umbrella API only when you already have a real operator bearer key or an OAuth-capable host that can complete delegated auth.
4. `GET /api/consumer` is the authoritative shopper-agent discovery document for personal consumer umbrella runs. It lists the consumer-prefill and direct follow-up URL templates and makes clear that these flows do not require a Coverage Cat operator API key.
5. `GET /api/consumer/openapi.yaml` is the authoritative request/response schema for the shopper-agent umbrella handoff endpoints. Use `/api/agent` and `/api/agent/openapi.yaml` only when you intentionally need delegated operator endpoints or read-only tool APIs.
6. Treat the OpenAPI and discovery documents as the source of truth when the skill prose and your memory disagree.

## API Contract Quick Reference

Pin the stable wire contract with `Coverage-Cat-API-Version: v1` when you want a fully explicit request shape, and send `Idempotency-Key: ...` on write calls you may retry. Coverage Cat also echoes `X-Skill-Version: 1.0.0` so you can log which published runtime-contract revision served the response. Example consumer-prefill write:

```bash
curl -X POST https://www.coveragecat.com/api/consumer/umbrella/prefill \
  -H "Content-Type: application/json" \
  -H "Coverage-Cat-API-Version: v1" \
  -H "Idempotency-Key: consumer-umbrella-prefill-001" \
  -d '{"credit_consent_pending":true,"intake":{"full_name":"Taylor Example"}}'
```

Sandbox and errors: set top-level `sandbox: true` only on the first delegated create call for a rehearsal `uid`; the consumer-prefill handoff does not use a separate sandbox flag. The common workflow-facing `error` values are `invalid_request | not_found | conflict | rate_limited | idempotency_conflict | stale_token | sandbox_unsupported | credit_consent_required | internal_error`; keep `message` for display or logging context.

Coverage Cat rotates a fresh `intake_access_token` in every successful direct follow-up response on the consumer-prefill path. Always reuse the newest token you have for the next `/api/intake/:uid/...` call.

Payment progression on the delegated path stays in `status`: `chosen -> documents_needed -> documents_pending_review -> payment_needed -> waiting_on_carrier -> bound`. When `payment_needed` is returned, Coverage Cat also returns `payment_url`.

## Choose the Path

This skill supports two different jobs. Pick one path first, because the consumer-prefill handoff and the operator-partner path do not share the same loop.

1. Use the consumer-prefill path when the shopper's own AI agent can gather facts from their vault, prior messages, or connected files before handing them to Coverage Cat.
2. When your runtime is on the default consumer MCP and no operator bearer key is present, stay on the consumer-operated subset from Path 1: `umbrella_consumer_prefill`, `consumer_intake_issues`, `consumer_intake_patch`, `umbrella_consumer_select`, `umbrella_consumer_bind`, `umbrella_consumer_attach`, and `umbrella_consumer_payment`.
3. Path 1 can stay on that consumer-operated subset through quoted offers, declarations upload, `payment_needed`, optional agentic payment collection, and `payment_url` fallback; do not switch to delegated tools just to reach checkout.
4. Use the operator-partner path only when you have a real Coverage Cat operator bearer key and approved back-office context you can use to prefill the application.
5. Do not mix the two paths in one session. Path 1 starts with an unauthenticated prefill call, then uses the returned `intake_access_token` for direct follow-up if your runtime can stay in chat. Path 2 uses the delegated operator API.

Coverage Cat also exposes read-only calculator and finder APIs. These are information tools, not purchase flows:

- `POST /api/agent/calculators/home`
- `POST /api/agent/calculators/umbrella`
- `POST /api/agent/calculators/auto-estimates`
- `POST /api/agent/calculators/collision-and-comprehensive-coverage`
- `POST /api/agent/calculators/file-a-claim`
- `POST /api/agent/tools/homeowners-agents/search`

Use these endpoints when the user asks for estimates, claim/coverage modeling, or licensed homeowners agents. Do not route umbrella shopping or buying requests here. They are rate limited more tightly than the purchase API, so cache the answer in the conversation and do not poll them repeatedly with the same inputs.

## Golden Paths

### Path 1: Consumer prefill handoff

1. Use this path when the shopper wants their own AI agent to gather the fullest umbrella application from user-controlled context before a single review step, or when you do not have an operator key.
2. On a cold start, do not open with a questionnaire. Call `POST /api/consumer/umbrella/prefill` first with `credit_consent_pending: true` and the fullest `intake` or estimate you can assemble. On the very first consumer-prefill handoff, prefer sending only `intake`; add top-level `field_estimates` only when you are certain each `field` already matches a current umbrella review field path.
   Use canonical consumer umbrella keys such as `full_name`, `email`, `phone_number`, `birthday`, `marital_status`, `license_state`, `license_number`, `occupation`, `line_of_work`, `spouse_full_name`, `spouse_birthday`, `spouse_license_state`, `spouse_license_number`, `address.{street,city,state,zip}`, `approximate_asset_value` or `net_worth_numeric`, `desired_coverage_limit`, `required_coverage_e_limit`, `vehicle_count` or `umbrella_details.motorized_vehicle_count`, `motorcycles`, `antique_vehicles`, `jet_skis`, `large_watercraft`, `property_count`, `non_us_property_count`, `current_auto_coverages`, `professional_entertainer_or_public_figure`, `household_open_claim`, `liability_loss_in_last_5_years`, `liability_loss_over_50k`, `four_or_more_losses_over_25k`, `reckless_driving_conviction`, and `felony`.
   Avoid non-canonical consumer keys such as `street`, `city`, `state`, `zip`, `drivers_license_state`, `drivers_license_number`, `net_worth`, or `desired_umbrella_liability_limit`.
   Do not send `vehicles` rows before submission. Use `vehicle_count` or `umbrella_details.motorized_vehicle_count` for the household vehicle count; actual vehicle details are collected later only if the selected offer requires them, such as a Markel bind follow-up.

   Example initial consumer-prefill payload:

   ```json
   {
     "credit_consent_pending": true,
     "intake": {
       "full_name": "Taylor Example",
       "email": "taylor@example.com",
       "marital_status": "married",
       "occupation": "product designer",
       "line_of_work": "software",
       "license_state": "CA",
       "license_number": "D0000001",
       "spouse_full_name": "Avery Example",
       "spouse_birthday": "1988-08-09",
       "spouse_license_state": "CA",
       "spouse_license_number": "A0000001",
       "address": {
         "street": "742 Evergreen Terrace",
         "city": "San Francisco",
         "state": "CA",
         "zip": "94110"
       },
       "approximate_asset_value": "$2.5M",
       "desired_coverage_limit": 2500000,
       "required_coverage_e_limit": 300000,
       "vehicle_count": 2,
       "motorcycles": 0,
       "antique_vehicles": 0,
       "jet_skis": 0,
       "large_watercraft": 0,
       "property_count": 1,
       "non_us_property_count": 0,
       "current_auto_coverages": "unsure",
       "professional_entertainer_or_public_figure": false,
       "household_open_claim": false,
       "liability_loss_in_last_5_years": false,
       "liability_loss_over_50k": false,
       "four_or_more_losses_over_25k": false,
       "reckless_driving_conviction": false,
       "felony": false
     }
   }
   ```
3. Keep the returned `uid`, `intake_access_token`, and `polling_token`. Treat `review_summary` as a checkpoint snapshot, not the final decision about whether the human needs to be interrupted yet. Until a later follow-up response shows quote progress, say explicitly that the application is not submitted yet and Coverage Cat has not received a submitted application yet. If it already includes a `review` preview plus a bundled `next_question` for shopper-owned details such as full name, email, phone, full address, birthday, marital status, driver's license state, driver's license number, and net worth, use that as the one review turn rather than falling back to a one-field questionnaire. Ask only for the explicitly missing shopper-owned fields staged there, and keep already reviewed values in the review section for corrections-only instead of asking the shopper to re-key them. If marital status means Coverage Cat still needs spouse or partner identity or license fields, keep spouse full name, spouse date of birth, spouse driver's license state, and spouse driver's license number in that same review turn instead of opening a second follow-up. If Coverage Cat says it reused a recent in-progress umbrella draft for the same email address, tell the shopper the reviewed values may come from that earlier unfinished Coverage Cat application. Do not describe those reused values as `historical_mode` unless the matching `field_estimates` row actually says so. When you summarize the gathered details or remaining items, use short labeled bullets or sections rather than a prose paragraph. When any shown value is estimated, mark that bullet or value with `*`, include the short note `* = estimated` once above and once below the list, and do not prefix every estimated line with `[Estimated]`.
4. If your runtime can continue in chat, call `GET /api/intake/:uid/issues` with the latest `Authorization: Bearer <intake_access_token>` and keep the browser closed for as long as Coverage Cat is still returning structured review, quote, or post-choose follow-up data.
5. Use `resume_url` only as the browser fallback when your runtime cannot continue in chat or the user explicitly wants Coverage Cat's UI on `/umbrella?resume=...`.
6. Optionally poll `GET /api/consumer/status?token=...` with the returned `polling_token` for coarse non-PII progress and quote highlights after the handoff.
7. If your runtime cannot prefill, fall back to the current environment origin plus `/umbrella` and stop there instead of imitating the delegated operator loop.

#### Direct follow-up API for Path 1

Use the `uid` plus `intake_access_token` returned by consumer prefill.

1. Call `GET /api/intake/:uid/issues` with the latest `Authorization: Bearer <intake_access_token>` to inspect what still blocks submission, any `pending_fields`, any estimated `field_estimates`, and the final `review` JSON once the intake is ready. Every successful response rotates a fresh `intake_access_token`; replace the old one immediately.
2. Use `PATCH /api/intake/:uid` with that same bearer token only for pre-submit review corrections, canonical umbrella count corrections such as `umbrella_details.motorized_vehicle_count`, optional top-level `field_estimates[]` rows whose `field` paths you are certain match the current umbrella review, and the final submit step. Never send `vehicles` rows on this pre-submit patch endpoint. Do not overload this patch endpoint with offer selection or post-choose bind fields.
3. Keep the human out of the loop until Coverage Cat has either reached `ready_for_submission` or returned a staged `review` preview plus a bundled `next_question` for the final shopper-owned details. If Coverage Cat still returns other `pending_fields` or a narrower `next_question`, keep filling them from user-controlled context or your own reasoning first. The intended UX is that the user sees one final completed review step, not a questionnaire.
4. On that single review turn, render `resource.review` as short labeled bullets or sections, not as a prose paragraph. Start by saying explicitly that the application is not submitted yet and Coverage Cat has not received a submitted application yet. If `next_question` is also present there, ask once only for the explicitly missing shopper-owned fields in it in plain English and list those requested items as bullets. Keep already reviewed values in the review section and invite corrections there instead of asking the shopper to re-key them. When any shown value is estimated, mark that bullet or value with `*`, include the short note `* = estimated` once above and once below the list, and do not prefix every estimated line with `[Estimated]`. When that missing-fields bundle covers full name, email, phone, full address, birthday, marital status, driver's license state, driver's license number, and net worth, collect those together in the same message as any free-text corrections. If `next_question` also includes spouse_full_name, spouse_birthday, spouse_license_state, or spouse_license_number because marital status requires them, collect that spouse bundle in the same review message too instead of opening a second follow-up. If the user confirms the application in that same reply, send one final `PATCH /api/intake/:uid` carrying those edits plus `confirm_submission: true`. Preliminary umbrella quotes do not require credit consent; consent for a soft credit check is collected after the user chooses an offer.
5. After submit, continue polling `GET /api/intake/:uid/issues` with the latest bearer token. While Coverage Cat returns `pending_quotes`, tell the shopper the application is submitted and carriers are still processing, keep polling, and do not present a partial quote set yet. Once Coverage Cat finishes the current speculative carrier checks, this endpoint returns structured umbrella offers for pre-choose review in chat, including per-offer `selection_token` values, per-offer `sample_policy_url` links when Coverage Cat has specimen policies on file, and detailed post-choose status payloads. Do not collapse that list down to only the recommended default offer when alternatives are present. Render every returned offer in a single markdown table with columns `Carrier | Coverage limit | Annual price | Min. auto limits | Notes` (one row per offer, recommended first, `(Recommended)` in the Notes column of the recommended row), paste the `sample_policy_url` into that row's Notes column as a markdown link when it is present, and never restate the same offer as a prose bullet. Below the table, add a `**Next steps to purchase:**` bulleted checklist of shopper action items only. The shopper should be able to answer that checklist in one reply with the chosen offer, any Min. auto limits confirmation, and `Yes` for the soft credit check if they are ready to buy. If `selection_guidance.same_reply_requirements` is present for an offer, add a conditional checklist bullet so the shopper can answer those listed items in that same selection reply. If Monoline or Markel is present, keep the declarations upload as the action bullet and move the 5-10 minute checkout timing into a separate note instead of the checklist.
6. When the user chooses an offer, call `POST /api/intake/:uid/select` with the latest bearer token, `selection_token`, `selection_confirmed: true`, and the real user's affirmative `credit_consent` only when Coverage Cat asks for it. If the quoted payload exposed `selection_guidance.same_reply_requirements` for that offer, send those answers under `intake` on this first `select` call so the shopper can answer choice + limits + consent + those carrier-specific items in one reply. If Coverage Cat still responds with `credit_consent_required` and also returns `next_question`, ask for both the consent for a soft credit check and those bind details in one shopper-facing turn, then retry `select` with both `credit_consent` and the bind-stage answers under `intake`. Successful responses return detailed statuses such as `needs_more_info_to_bind`, `documents_needed`, `payment_needed`, `waiting_on_carrier`, or `bound`, plus a fresh token for the next follow-up call.
7. If `select` or `issues` returns `needs_more_info_to_bind` or `ready_to_finalize`, call `POST /api/intake/:uid/bind` with the latest bearer token and only the current `next_question` fields inside `intake`. This later bind-stage follow-up is where actual vehicle details may be requested for selected offers such as Markel; do not move that request into the pre-submit review. When the response says `ready_to_finalize`, call `bind` again with the same `uid` and no `intake` patch.
8. If `select`, `bind`, or `issues` returns `documents_needed`, upload declarations through `POST /api/intake/:uid/attach` with the latest bearer token. You may send one file with `filename`, base64 `content`, and `type` in `declarations | auto_declarations`, or send multiple files together in one call with `attachments[]`. If both the home and auto declarations are already available, prefer one attach call so the shopper only has to approve one upload turn. Use `attach` only after selection and only when Coverage Cat explicitly asks for documents.
9. If Coverage Cat returns `payment_needed`, first call `POST /api/intake/:uid/payment` with `payment_collection_requested: true` when your runtime can securely collect payment details. Render the returned Stripe setup details, then call the same endpoint again with `setup_intent_id` and `payment_method_id` after the shopper completes the secure form. If your runtime cannot securely collect payment details, share `payment_url` and let Coverage Cat's browser handle secure payment or carrier e-sign. Keep `GET /api/intake/:uid/issues` as the structured polling loop for post-choose processing, `waiting_on_carrier`, and `bound`.
10. Keep `GET /api/consumer/status?token=...` as the coarse fallback if you only need non-PII progress.

### Path 2: Operator-partner delegated flow

1. Get or reuse an operator key.
2. Call `draft` with the fullest intake and any matching `field_estimates`.
3. Do not show `needs_more_info` to the user yet. Keep searching approved context for the missing facts.
4. If you are rehearsing, set top-level `sandbox: true` on that very first `draft` call. That `uid` stays sandbox-scoped, `quotes` returns mocked offers, and `status` stays mocked without live customer email or carrier traffic.
5. When Coverage Cat returns `ready_for_review`, render the review for the user as short labeled bullets or sections sourced from the review JSON. When any shown value is estimated, mark that bullet or value with `*`, include the short note `* = estimated` once above and once below the list, and do not prefix every estimated line with `[Estimated]`.
6. After the user confirms the review, call `quotes`. If it returns `pending_quotes`, keep polling `status` until Coverage Cat finishes the current speculative carrier checks, and do not present partial offers. If it returns `quoted`, present every returned offer. Then continue through `select` (which collects the credit-consent prompt) and `status`. Use `bind` only if Coverage Cat asks for bind-stage fields.

### Read-only modeling tools

1. Use the calculator and finder endpoints when the user wants estimates, claim math, or agent search, not a delegated umbrella purchase.
2. Cache the result in the conversation because these tools are rate limited more tightly than the purchase APIs.
3. Do not open a delegated purchase session unless the user wants Coverage Cat to quote or continue a live umbrella application.

## Authentication

Path 1, the consumer-prefill handoff, does not use an operator bearer key for the initial prefill call. Coverage Cat returns a rotating `intake_access_token` that the agent should send back as `Authorization: Bearer <intake_access_token>` on `GET /api/intake/:uid/issues`, `PATCH /api/intake/:uid`, `POST /api/intake/:uid/select`, `POST /api/intake/:uid/bind`, and `POST /api/intake/:uid/attach`. Always reuse the newest token returned by the latest direct follow-up response. Path 2, the delegated operator loop, uses an operator bearer key.

Do not ask a personal shopper for a Coverage Cat operator API key or delegated OAuth just to run Path 1. Stay on the consumer MCP or the shopper-safe REST surface for consumer-prefill and direct follow-up.

Use an operator-issued bearer token for every delegated umbrella endpoint on Path 2.

1. Request a 6-digit code with `POST /api/agent/key/request` and `{"email":"operator@example.com"}`.
2. Confirm it with `POST /api/agent/key/confirm` and `{"email":"operator@example.com","otp":"123456"}`. If you want delegated-umbrella customer emails copied to a runtime assistant mailbox, include optional registration data such as `assistant_email`.
3. Save the returned bearer token and send it on all subsequent requests as `Authorization: Bearer <token>`.

The legacy field name `code` is still accepted by `/api/agent/key/confirm`, but new integrations should send `otp`.

## Base URL

Use the current environment origin for every request.

- Production examples in these docs use `https://www.coveragecat.com`
- Local development often uses `http://localhost:4000`

On local development instances that expose Swoosh's mailbox preview, OTP emails are visible at `/dev/mailbox`.

## Goal

Keep the conversation short, safe, and user-led:

1. Pick the path first. Use the consumer-prefill handoff when you are working from the user's own context, and the delegated loop when you have an operator bearer key plus approved back-office context.
2. Search that available context first and assemble the fullest umbrella application you can before involving the human.
3. Keep the returned `uid`, `intake_access_token`, and `polling_token` together; they are the full consumer handoff state for chat continuation. Replace `intake_access_token` whenever a direct follow-up response rotates it.
4. On delegated umbrella draft calls, send non-user-confirmed values in `intake` and attach matching `field_estimates` metadata so Coverage Cat can persist provenance and mark them in review. On the initial consumer umbrella prefill handoff, prefer sending only `intake`; add top-level `field_estimates` only when you are certain each `field` already matches a current umbrella review field path.
5. Keep `needs_more_info` behind the scenes when you can. The intended UX is that the human sees only the completed review page, or at most one staged review turn that also gathers the final shopper-owned details bundle.
6. On every pre-submit user-facing turn, explicitly say the application is not submitted yet and Coverage Cat has not received a submitted application yet.
7. When Coverage Cat returns `ready_for_review`, or when a consumer follow-up payload carries a staged `review` plus bundled shopper-detail `next_question`, render the completed application as short labeled bullets or sections sourced from the review JSON. When any shown value is estimated, mark that bullet or value with `*`, include the short note `* = estimated` once above and once below the list, and do not prefix every estimated line with `[Estimated]`. If that staged bundle includes spouse or partner identity or license fields because marital status requires them, keep those spouse fields in the same single review turn instead of creating a second questionnaire. Do not ask for credit consent yet — it is collected after the user chooses an offer.
8. Treat any free-text corrections during review as edits to patch back through the follow-up API or delegated `draft`, depending on the path.
9. Present every returned offer as a single markdown table (`Carrier | Coverage limit | Annual price | Min. auto limits | Notes`), never a scattered list of prose sentences or a shortlist teaser. When an offer includes `sample_policy_url`, paste it directly into that row's Notes column as a markdown link so the shopper can inspect the policy before choosing. Below the table, always render a `**Next steps to purchase:**` bulleted checklist of shopper action items only, keep timing notes outside that checklist, and make the selection turn answerable in one reply with the offer choice, any Min. auto limits confirmation, and `Yes` for the soft credit check when the shopper is ready to buy.
10. At `select`, accept only a real-user `Yes` to the consent for a soft credit check prompt before binding a chosen offer.
11. Keep the bind follow-up in chat whenever possible.
12. For Monoline and Markel, collect declarations first. Once Coverage Cat has those uploads on file, it usually returns the final price and secure checkout within about 5-10 minutes while declarations verification continues in parallel.
13. If any response includes `sandbox: true`, treat every offer, token, and link as mocked test data and do not continue into a live customer handoff.

## Conversation Rules

- Lead with a compact framing statement such as: "I can help compare home and umbrella options. If you want umbrella coverage through Coverage Cat, I can also help complete the application and selection flow."
- Pick the path first. Use the consumer-prefill handoff when no operator bearer key is present, and use the delegated operator loop only when a real key is already available.
- Do not use this umbrella workflow for homeowners purchase. Use the dedicated homeowners purchase skill instead.
- Reuse known facts and search the available context before asking the human anything: user-controlled vaults, prior messages, connected files, CRM records, prior Coverage Cat sessions, email threads, document drives, and OCR'd policy documents.
- On a cold start, do not open by asking for name, email, phone, full address, birthday, marital status, driver's license number, or net worth. Read the machine-readable docs, call consumer prefill first with the fullest estimate you can justify, then use Coverage Cat's staged review turn to collect the shopper-owned bundle once.
- Do not re-ask fields already present in `known_summary` unless the user wants to change them.
- Do not surface `needs_more_info` as a step-by-step questionnaire unless your product intentionally falls back to one after exhausting operator-side context. The intended umbrella UX is a single review page followed by offers.
- For consumer umbrella handoffs, use canonical keys such as `address.{street,city,state,zip}`, `license_state`, `license_number`, `approximate_asset_value` or `net_worth_numeric`, `desired_coverage_limit`, `required_coverage_e_limit`, and `vehicle_count` or `umbrella_details.motorized_vehicle_count` for umbrella counts. Avoid top-level `street`/`city`/`state`/`zip`, `drivers_license_state`, `drivers_license_number`, `net_worth`, or `desired_umbrella_liability_limit`.
- On every pre-submit user-facing turn, say explicitly that the application is not submitted yet and Coverage Cat has not received a submitted application yet.
- When you list gathered details, estimated answers, or remaining items for the shopper, use short labeled bullets or sections rather than a prose paragraph.
- When any shown value is estimated, mark that bullet or value with `*`, include the short note `* = estimated` once above and once below the list, and do not prefix every estimated line with `[Estimated]`.
- If Coverage Cat stages a review plus shopper-detail `next_question`, show the assembled review and ask only for the explicitly missing shopper-owned fields together in one message, in plain English rather than field names. Keep already reviewed values in the review section for corrections-only instead of asking the shopper to re-key them. Keep the assembled review and the requested items in labeled bullets or sections. When it includes full name, email, phone, full address, birthday, marital status, driver's license state, driver's license number, and net worth, keep that as one turn instead of splitting it up. If marital status means Coverage Cat also asks for spouse or partner full name, spouse date of birth, spouse driver's license state, or spouse driver's license number, keep that spouse bundle in the same message too instead of opening a second follow-up. If Coverage Cat says it reused a recent in-progress umbrella draft for the same email address, explain that reviewed values may come from that earlier unfinished Coverage Cat application rather than from `historical_mode` unless the matching `field_estimates` row says so.
- On delegated umbrella draft calls, any value coming from CRM, documents, email threads, or heuristics that the human has not directly confirmed yet should be sent in `intake` with a matching `field_estimates` row carrying `field`, `source`, and `confidence`. On the initial consumer umbrella handoff, prefer sending only `intake`; add top-level `field_estimates` only when you are certain each `field` already matches a current umbrella review field path.
- Do not send `vehicles` rows before submission. Use `vehicle_count` or `umbrella_details.motorized_vehicle_count` for the household vehicle count, and if the staged review shows the wrong estimated umbrella count from context, patch the canonical `umbrella_details.*` count field before you surface the final review.
- Do not ask for credit consent on pre-submit review turns. Once the shopper is actively choosing an offer, you may collect the real user's `Yes` in that same reply so `select` can proceed without a second consent-only turn.
- When quoted offers include Monoline or Markel, explain during quote review that current home and auto declarations are still required before payment and that secure checkout is usually ready within about 5-10 minutes once those uploads are on file.
- Use `resume_url` only as a browser fallback. When Coverage Cat reaches `payment_needed`, prefer the chat-native `/payment` flow when your runtime can securely collect payment details; otherwise hand the user to the returned browser URL for payment or carrier-managed signing.
- In sandbox mode, use fake or test contact details and stop at `status`. Do not treat sandbox offers, tokens, or links as a real customer handoff.

## API Loop

### 1. Start or update the delegated session

Call `POST /api/agent/umbrella/draft`.

- Omit `uid` to create a new session.
- Start with the fullest `intake` patch you can build from approved operator-side context, not just the fields the human typed in the current turn.
- Include `field_estimates` for every value you are sending in `intake` that is not already directly user-confirmed. Omit `field_estimates` only for user-confirmed values.
- Send only changed fields on follow-up calls.
- If Coverage Cat reuses a recent delegated draft for the same buyer email, keep working on the returned `uid` and `review_token`; that response is continuing one existing session, not creating a second application.
- Accepted aliases inside `intake`:
  Send `birthday` as the applicant birth date and Coverage Cat maps it to internal `age`.
  Send `spouse_birthday` and Coverage Cat maps it to `spouse_age`.
  Send `approximate_asset_value` and Coverage Cat maps it to `net_worth_numeric`.
- Send birthdays to Coverage Cat as ISO dates such as `1990-04-01`. If the shopper replies in normal US `MM-DD-YYYY` form such as `04-01-1990`, normalize it to ISO before you patch Coverage Cat. Do not send raw integer ages.
- If you are rehearsing the delegated umbrella flow, set top-level `sandbox: true` on this first create call only.

Example first call when CRM/email/docs already provide nearly everything:

```json
{
  "intake": {
    "full_name": "Taylor Agent",
    "email": "taylor.agent@example.com",
    "phone_number": "+12065550100",
    "birthday": "1990-04-01",
    "license_number": "WA1234567",
    "marital_status": "single",
    "address": {
      "street": "123 Main St",
      "city": "Seattle",
      "state": "WA",
      "zip": "98101"
    }
  },
  "field_estimates": [
    {
      "field": "marital_status",
      "source": "crm",
      "confidence": "medium"
    }
  ]
}
```

Example first sandbox rehearsal call:

```json
{
  "sandbox": true,
  "intake": {
    "full_name": "Taylor Sandbox",
    "email": "sandbox-umbrella@example.test",
    "phone_number": "+12065550100",
    "birthday": "1990-04-01",
    "license_number": "WA1234567",
    "marital_status": "single",
    "address": {
      "street": "123 Main St",
      "city": "Seattle",
      "state": "WA",
      "zip": "98101"
    }
  }
}
```

If the response is:

- `needs_more_info`: do not show this incomplete state to the human yet. Use `pending_fields`, `suggested_values`, and `next_question` to keep searching the operator's allowed context. The intended flow is to keep calling `draft` until Coverage Cat can return a completed review page.
- If you exhaust approved operator-side context and still get `needs_more_info`, stop and escalate or hand off rather than drifting into a long questionnaire unless your product explicitly chooses that fallback.
- `ready_for_review`: render the single review page described below.
- `ineligible`: tell the user Coverage Cat cannot complete delegated umbrella purchase for that state yet.
- `409 conflict`: Coverage Cat found a more advanced recent delegated umbrella session for that buyer email. Reuse the returned `uid` and continue from its current `status` instead of retrying `draft`.

If the response also includes `sandbox: true`:

- Treat the session as mocked test data.
- Keep the same `uid` through `quotes` and `status`.
- Do not continue into `select`, `bind`, or `attach` on that sandbox `uid`.

### 2. Review page

When Coverage Cat returns `ready_for_review`:

- Render the `review` object as short labeled bullets or sections rather than a prose paragraph.
- Use `field_estimates` to mark estimated values with `*` inside that JSON, include the short note `* = estimated` once above and once below the list, and do not prefix every estimated line with `[Estimated]`.
- Tell the user they only need to confirm the reviewed application and send any corrections in free text.
- If the user edits anything, call `draft` again with those edits, omit those corrected fields from `field_estimates` unless they are still estimated on your side, and wait for a new `review_token`.
- Do not ask for credit consent yet. Preliminary quotes do not require it; Coverage Cat collects consent for a soft credit check after the user chooses an offer.
- Only proceed once the user has explicitly confirmed the reviewed application is correct.

### 3. Quote step

Call `POST /api/agent/umbrella/quotes` with:

- `uid`
- `review_token`
- `review_confirmed: true`

Rules:

- Preliminary umbrella quotes do not require credit consent. If the application changed, go back to `draft` and wait for a fresh `review_token`.
- If this is a sandbox session, the returned offers are mocked and later live-continuation endpoints are intentionally unavailable.

If the response is `pending_quotes`:

- Tell the user Coverage Cat is still generating quotes and will publish the full set together once the current carrier checks finish.
- Poll `status`.
- Do not present partial umbrella offers while the status is still `pending_quotes`.

If the response is `quoted`:

- Lead with `response_summary`.
- Render offers as a **single markdown table** with these columns in this order: `Carrier | Coverage limit | Annual price | Min. auto limits | Notes`. One row per offer, sorted by rank (recommended first). Put `(Recommended)` in the Notes column for the recommended offer. Leave Min. auto limits blank when an offer has no `minimum_auto_limits` value. When an offer includes `sample_policy_url`, paste it directly into that row's Notes column as a markdown link such as `[Sample policy](...)`. Do NOT render offers as a scattered bullet list of prose sentences like "Carrier X — $941/yr for $2M" and do NOT restate offer prices or coverage limits in the surrounding prose — the table is the source of truth.
- Format prices as `$941/yr` and coverage limits as `$2M` / `$3M` (do not spell out "per year" or "million").
- Put every returned offer in the table, even when more than three are present. Do not replace the full list with a shortlist or an `N more offer(s) available` teaser line.
- After the table, render a labeled section titled exactly `**Next steps to purchase:**` as a bulleted checklist of shopper action items only. Never bury these in a paragraph, and keep timing or advisory notes outside the checklist. Include only bullets that apply to the offers shown:
  - Always include: `Reply with the offer you want by carrier + coverage or row number.`
  - When any shown offer has `minimum_auto_limits`, include: `In that same reply, confirm your auto policy already meets the Min. auto limits shown for the offer you pick, or say it needs to be raised first.`
  - Always include: `If you're ready to buy, include Yes to allow Coverage Cat's soft credit check in that same reply.`
  - When any shown offer is Monoline or Markel, include: `If you pick Monoline or Markel, upload current auto and home declarations pages (PDF or photo) after selection so Coverage Cat can prepare secure checkout.`
- After the checklist, if any shown offer is Monoline or Markel, add a separate note such as `Timing note: secure checkout is usually ready within about 5-10 minutes after Coverage Cat receives those uploads.`
- Use `selection_guidance` to explain the suggested default choice.
- If `selection_guidance.same_reply_requirements` lists an offer, surface those listed items as a short conditional checklist bullet so the shopper can answer them in the same selection reply instead of waiting for a later bind-only turn.
- Make clear that premiums are estimates until final underwriting.
- If the response also includes `sandbox: true`, stop at display/testing: do not call `select`, `bind`, or `attach` on this `uid`.

If the response is `no_quotes`:

- Tell the user Coverage Cat finished checking the available umbrella carriers and did not find an offer for this profile right now.
- Offer the secure `compare_details_url` if they want to continue on Coverage Cat directly.

### 4. Selection step

After the user chooses an offer and confirms that choice, call `POST /api/agent/umbrella/select` with:

- `uid`
- `selection_token`
- `selection_confirmed: true`
- `credit_consent.answer: "Yes"`
- `credit_consent.collected_from_user: true`

Rules for the consent block:

- The consent answer must come from the real user, not from you. This is the soft credit pull that carriers require to bind the selected offer.
- Only proceed on an explicit `Yes`. If the user says anything else, stop and explain that Coverage Cat cannot bind an offer without the credit-pull authorization.
- If Coverage Cat returns `credit_consent_required` because consent was not sent or was declined, render `credit_consent_question` for the real user. If `next_question` is also present, ask for the consent and every listed bind detail in the same shopper-facing turn, then retry `select` with the same `selection_token`, the user's `Yes`, and those `intake` answers.

Do not call this step for sandbox sessions. Sandbox umbrella currently stops at mocked `quotes` and `status`.

Handle the response like this:

- `credit_consent_required`: if `next_question` is absent, ask `credit_consent_question` now and retry `select` with the same `selection_token` plus the user's affirmative consent. If `next_question` is present, ask for the consent and every listed bind detail in the same shopper-facing turn, then retry `select` with both the affirmative consent and those `intake` answers.
- `needs_more_info_to_bind`: keep the user in chat. Ask only the fields in `next_question`, and also offer `offers_url` so the user can finish that same step inside Coverage Cat's GUI if they prefer. Then call the bind endpoint.
- `documents_needed`: Coverage Cat accepted the selection and now needs the user's current home and auto declarations before it can finish processing the offer and prepare secure checkout. Upload them through `attach` if you already have them in context. Otherwise, ask the user to share them, and also offer `offers_url` so they can complete the upload in Coverage Cat's GUI if that is easier.
- `documents_pending_review`: Coverage Cat has the declarations on file and is processing the chosen offer now. The final price and secure checkout page are usually ready within about 5-10 minutes. Poll `status`.
- When `documents_pending_review` includes `poll_after_seconds`, wait that long before polling again unless the user asks for an update sooner.
- `payment_needed`: Coverage Cat has the declarations on file and secure payment setup is ready. If your runtime can securely collect payment details, use `POST /api/intake/:uid/payment`; otherwise share `payment_url`, then poll `status`.
- `chosen`: Coverage Cat has accepted the selection and handed the case to the carrier's normal follow-up path. This is the expected immediate post-select state for RLI and similar carrier-managed flows.

### 5. Bind continuation step

If `select` or `status` says the offer `needs_more_info_to_bind`, call `POST /api/agent/umbrella/bind`.

- Include `uid`.
- Include a minimal `intake` patch with only the fields from the current bind-stage `next_question`.
- Do not send quote-affecting edits here. If the user wants to change material application data, go back to the draft/review flow instead.

Do not call this step for sandbox sessions. Sandbox umbrella currently stops at mocked `quotes` and `status`.

Handle the response like this:

- `needs_more_info_to_bind`: ask only the next bind-stage question bundle, offer `offers_url` so the user can finish it in Coverage Cat's GUI if they prefer, then call `bind` again.
- `ready_to_finalize`: call `bind` again with the same `uid` and no `intake` patch.
- `documents_needed`: upload declarations or ask the user to provide them, and also offer `offers_url` so they can complete the upload in Coverage Cat's GUI. Then poll `status`.
- `documents_pending_review`: tell the user Coverage Cat is processing the chosen offer now and secure checkout is usually ready within about 5-10 minutes, then poll `status`.
- Use `poll_after_seconds` from the API response when present. If the status remains pending, tell the user processing is still in progress. There is no extra customer action unless Coverage Cat asks for updated documents.
- `payment_needed`: if your runtime can securely collect payment details, use `POST /api/intake/:uid/payment`; otherwise share `payment_url` and ask the user to complete secure payment setup there.
- `chosen`: keep the user updated through `status`. Share `offers_url` if they want Coverage Cat's offers page for review.

#### Declarations pages during bind

For Monoline and Markel, proactively collect the user's current auto and home declarations pages once the selected offer reaches `documents_needed`.

- If you have access to the user's email, file storage, or photos, search for these documents first (look for PDFs or images with terms like "declarations", "dec page", "policy summary", or the carrier name).
- If you find candidates, confirm with the user before using them.
- If you cannot find them automatically, ask the user to share or photograph them.
- Forward any documents you obtain via `POST /api/agent/umbrella/attach` so Coverage Cat can process the chosen offer and prepare checkout. If both the home and auto declarations are already available, prefer one attach call with `attachments[]`.
- `attach` is only for Monoline or Markel after an offer has been selected. It is not a generic file-upload endpoint for pre-quote drafts.

### 6. Status step

Use `POST /api/agent/umbrella/status` to keep the user in chat after selection.

For sandbox sessions, use `status` only to rehearse the mocked draft/review/quote loop. If the payload includes `sandbox: true`, keep treating every link, offer, and next step as test data.

Interpret the main statuses like this:

- `documents_needed`: Coverage Cat needs declarations uploads before payment. Upload any files you already have, or ask the user to provide them, and offer `offers_url` so they can complete the upload in Coverage Cat's GUI if they prefer.
- `documents_pending_review`: Coverage Cat has the declarations on file and is processing the chosen offer now. Tell the user secure checkout is usually ready within about 5-10 minutes.
- If the API returns `poll_after_seconds`, use that as the default poll cadence. If the status stays pending, keep the user informed that processing is still in progress.
- `payment_needed`: Coverage Cat has the declarations on file and is ready for payment collection. If your runtime can securely collect payment details, use `POST /api/intake/:uid/payment`; otherwise share `payment_url`.
- `waiting_on_carrier`: no user action is needed. Tell the user Coverage Cat is waiting on the carrier's follow-up path. Offer `offers_url` if they want to review the selected offer. For agency-pay carriers, this means declarations uploads and payment are already on file. For RLI-style flows, any remaining signing or payment link comes from the carrier.
- `bound`: tell the user the policy is bound.
- `info_requested`: Coverage Cat has a manual follow-up request. Keep the user in chat if you can, and only use the fallback link if you need the exact human-authored request.
- `blocked`: Coverage Cat hit a carrier-side block. Do not re-collect the same information again unless Coverage Cat asks for it.

## Security Rules

- Never answer the credit-consent question on the user's behalf.
- Never fabricate review confirmation or selection confirmation.
- Treat `uid`, `review_token`, `selection_token`, and secure URLs as sensitive session data.
- Do not paste tokens into unnecessary summaries or logs.
- If Coverage Cat says the review or selection is stale, go back to the earlier step and get a fresh token.
- Do not expose raw `needs_more_info` payloads to the user as if they were the intended umbrella UX. Keep operator-side enrichment separate from the human review page when you can.
- During bind continuation, only send the fields explicitly requested in the current `next_question`.
- Once a session has moved into bind or post-select checkout, do not call `quotes` again with an old `review_token`. Continue from `bind` or `status`, or restart from `draft` if the application materially changed.

## HCI and Token-Efficiency Rules

- Prefer one concise question bundle over many tiny turns, but only within the current `next_question`.
- Use the API to discover missing fields instead of dumping the full umbrella questionnaire on the user.
- Prefer operator-side enrichment plus one completed review page over step-by-step human form fill.
- Reflect back what Coverage Cat already knows before asking for new facts.
- When `ready_for_review` is returned, render the JSON review directly. The credit-consent prompt comes later, at `select`.
- When presenting offers, keep the first pass short: recommended offer first, then alternatives.
- Keep the user in chat after selection by polling `status`. Use browser handoff only for secure payment setup or manual fallback cases where Coverage Cat has not exposed structured chat data yet.
