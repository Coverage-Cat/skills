---
name: coverage-cat-umbrella-purchase
description: "Use whether you're a consumer's own AI agent or a partner operator. This skill covers two umbrella paths: a consumer-prefill handoff that assembles one review page from the user's own context, keeps quote review and post-choose follow-up in chat through the direct intake API when possible, and returns preliminary quotes without upfront credit consent, and an operator-partner delegated quote and bind loop that collects credit consent at selection. It also lists the read-only calculator and finder endpoints as separate, non-purchase tools."
---

# Coverage Cat Umbrella Purchase Skill

You are helping a user through Coverage Cat's umbrella purchase flow. This skill covers both the consumer-prefill handoff and the operator-partner delegated quote loop. Use the separate homeowners purchase skill for homeowners quoting.

## Machine-Readable Contract

Read Coverage Cat's machine-readable surfaces before you infer the endpoint map:

1. `GET /api/agent` lists the umbrella operation map (`draft`, `quotes`, `select`, `bind`, `status`, `attach`) with concrete URLs.
2. `GET /api/agent/openapi.yaml` is the authoritative request/response schema for the published umbrella and consumer handoff endpoints.
3. Treat the OpenAPI and discovery documents as the source of truth when the skill prose and your memory disagree.

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
2. Use the operator-partner path when you have a real Coverage Cat operator bearer key and approved back-office context you can use to prefill the application.
3. Do not mix the two paths in one session. Path 1 starts with an unauthenticated prefill call, then uses the returned `intake_access_token` for direct follow-up if your runtime can stay in chat. Path 2 uses the delegated operator API.

Coverage Cat also exposes read-only calculator and finder APIs. These are information tools, not purchase flows:

- `POST /api/agent/calculators/home`
- `POST /api/agent/calculators/umbrella`
- `POST /api/agent/calculators/auto-estimates`
- `POST /api/agent/calculators/collision-and-comprehensive-coverage`
- `POST /api/agent/calculators/file-a-claim`
- `POST /api/agent/tools/homeowners-agents/search`

Use these endpoints when the user asks for estimates, claim/coverage modeling, or licensed homeowners agents. They are rate limited more tightly than the purchase API, so cache the answer in the conversation and do not poll them repeatedly with the same inputs.

## Golden Paths

### Path 1: Consumer prefill handoff

1. Use this path when the shopper wants their own AI agent to gather the fullest umbrella application from user-controlled context before a single review step, or when you do not have an operator key.
2. On a cold start, do not open with a questionnaire. Call `POST /api/consumer/umbrella/prefill` first with the fullest `intake` or estimate you can assemble, any matching `field_estimates`, and `credit_consent_pending: true`.
3. Keep the returned `uid`, `intake_access_token`, and `polling_token`. Treat `review_summary` as a checkpoint snapshot, not the final decision about whether the human needs to be interrupted yet. Until a later follow-up response shows quote progress, say explicitly that the application is not submitted yet and Coverage Cat has not received a submitted application yet. If it already includes a `review` preview plus a bundled `next_question` for shopper-owned details such as full name, email, phone, full address, birthday, marital status, driver's license state, driver's license number, and net worth, use that as the one review turn rather than falling back to a one-field questionnaire. When you summarize the gathered details or remaining items, use short labeled bullets or sections rather than a prose paragraph.
4. If your runtime can continue in chat, call `GET /api/intake/:uid/issues` with the latest `Authorization: Bearer <intake_access_token>` and keep the browser closed for as long as Coverage Cat is still returning structured review, quote, or post-choose follow-up data.
5. Use `resume_url` only as the browser fallback when your runtime cannot continue in chat or the user explicitly wants Coverage Cat's UI on `/umbrella?resume=...`.
6. Optionally poll `GET /api/consumer/status?token=...` with the returned `polling_token` for coarse non-PII progress and quote highlights after the handoff.
7. If your runtime cannot prefill, fall back to the current environment origin plus `/umbrella` and stop there instead of imitating the delegated operator loop.

#### Direct follow-up API for Path 1

Use the `uid` plus `intake_access_token` returned by consumer prefill.

1. Call `GET /api/intake/:uid/issues` with the latest `Authorization: Bearer <intake_access_token>` to inspect what still blocks submission, any `pending_fields`, any estimated `field_estimates`, and the final `review` JSON once the intake is ready. Every successful response rotates a fresh `intake_access_token`; replace the old one immediately.
2. Use `PATCH /api/intake/:uid` with that same bearer token only for pre-submit review corrections, inferred umbrella values plus matching top-level `field_estimates[]` rows, and the final submit step. Do not overload this patch endpoint with offer selection or post-choose bind fields.
3. Keep the human out of the loop until Coverage Cat has either reached `ready_for_submission` or returned a staged `review` preview plus a bundled `next_question` for the final shopper-owned details. If Coverage Cat still returns other `pending_fields` or a narrower `next_question`, keep filling them from user-controlled context or your own reasoning first. The intended UX is that the user sees one final completed review step, not a questionnaire.
4. On that single review turn, render `resource.review` as short labeled bullets or sections, not as a prose paragraph. Start by saying explicitly that the application is not submitted yet and Coverage Cat has not received a submitted application yet. If `next_question` is also present there, ask once for every shopper-owned field in it in plain English and list those requested items as bullets. When that bundle covers full name, email, phone, full address, birthday, marital status, driver's license state, driver's license number, and net worth, collect those together in the same message as any free-text corrections. If the user confirms the application in that same reply, send one final `PATCH /api/intake/:uid` carrying those edits plus `confirm_submission: true`. Preliminary umbrella quotes do not require credit consent; the soft-credit pull authorization is collected after the user chooses an offer.
5. After submit, continue polling `GET /api/intake/:uid/issues` with the latest bearer token. Once Coverage Cat has the first batch of quotes, this endpoint returns structured umbrella offers for pre-choose review in chat, including per-offer `selection_token` values and detailed post-choose status payloads. Do not collapse that list down to only the recommended default offer when alternatives are present.
6. When the user chooses an offer, call `POST /api/intake/:uid/select` with the latest bearer token, `selection_token`, `selection_confirmed: true`, and the real user's affirmative `credit_consent` only when Coverage Cat asks for it. Successful responses return detailed statuses such as `needs_more_info_to_bind`, `documents_needed`, `payment_needed`, `waiting_on_carrier`, or `bound`, plus a fresh token for the next follow-up call.
7. If `select` or `issues` returns `needs_more_info_to_bind` or `ready_to_finalize`, call `POST /api/intake/:uid/bind` with the latest bearer token and only the current `next_question` fields inside `intake`. When the response says `ready_to_finalize`, call `bind` again with the same `uid` and no `intake` patch.
8. If `select`, `bind`, or `issues` returns `documents_needed`, upload declarations through `POST /api/intake/:uid/attach` with the latest bearer token, `filename`, base64 `content`, and `type` in `declarations | auto_declarations`. Use `attach` only after selection and only when Coverage Cat explicitly asks for documents.
9. If Coverage Cat returns `payment_needed`, share `payment_url` and let Coverage Cat's browser handle secure payment or carrier e-sign. Keep `GET /api/intake/:uid/issues` as the structured polling loop for declarations review, `waiting_on_carrier`, and `bound`.
10. Keep `GET /api/consumer/status?token=...` as the coarse fallback if you only need non-PII progress.

### Path 2: Operator-partner delegated flow

1. Get or reuse an operator key.
2. Call `draft` with the fullest intake and any matching `field_estimates`.
3. Do not show `needs_more_info` to the user yet. Keep searching approved context for the missing facts.
4. If you are rehearsing, set top-level `sandbox: true` on that very first `draft` call. That `uid` stays sandbox-scoped, `quotes` returns mocked offers, and `status` stays mocked without live customer email or carrier traffic.
5. When Coverage Cat returns `ready_for_review`, render the review for the user as short labeled bullets or sections sourced from the review JSON.
6. After the user confirms the review, call `quotes` and present the offers. Then continue through `select` (which collects the credit-consent prompt) and `status`. Use `bind` only if Coverage Cat asks for bind-stage fields.

### Read-only modeling tools

1. Use the calculator and finder endpoints when the user wants estimates, claim math, or agent search, not a delegated umbrella purchase.
2. Cache the result in the conversation because these tools are rate limited more tightly than the purchase APIs.
3. Do not open a delegated purchase session unless the user wants Coverage Cat to quote or continue a live umbrella application.

## Authentication

Path 1, the consumer-prefill handoff, does not use an operator bearer key for the initial prefill call. Coverage Cat returns a rotating `intake_access_token` that the agent should send back as `Authorization: Bearer <intake_access_token>` on `GET /api/intake/:uid/issues`, `PATCH /api/intake/:uid`, `POST /api/intake/:uid/select`, `POST /api/intake/:uid/bind`, and `POST /api/intake/:uid/attach`. Always reuse the newest token returned by the latest direct follow-up response. Path 2, the delegated operator loop, uses an operator bearer key.

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
4. Send non-user-confirmed values in `intake` and attach matching `field_estimates` metadata so Coverage Cat can persist provenance and mark them in review.
5. Keep `needs_more_info` behind the scenes when you can. The intended UX is that the human sees only the completed review page, or at most one staged review turn that also gathers the final shopper-owned details bundle.
6. On every pre-submit user-facing turn, explicitly say the application is not submitted yet and Coverage Cat has not received a submitted application yet.
7. When Coverage Cat returns `ready_for_review`, or when a consumer follow-up payload carries a staged `review` plus bundled shopper-detail `next_question`, render the completed application as short labeled bullets or sections sourced from the review JSON. Do not ask for credit consent yet — it is collected after the user chooses an offer.
8. Treat any free-text corrections during review as edits to patch back through the follow-up API or delegated `draft`, depending on the path.
9. Present structured offers clearly.
10. At `select`, accept only a real-user `Yes` to the credit-consent prompt before binding a chosen offer.
11. Keep the bind follow-up in chat whenever possible.
12. For Monoline and Markel, collect declarations first, wait for Coverage Cat to verify them, then share payment only after verification is complete.
13. If any response includes `sandbox: true`, treat every offer, token, and link as mocked test data and do not continue into a live customer handoff.

## Conversation Rules

- Lead with a compact framing statement such as: "I can help compare home and umbrella options. If you want umbrella coverage through Coverage Cat, I can also help complete the application and selection flow."
- Pick the path first. Use the consumer-prefill handoff when no operator bearer key is present, and use the delegated operator loop only when a real key is already available.
- Do not use this umbrella workflow for homeowners purchase. Use the dedicated homeowners purchase skill instead.
- Reuse known facts and search the available context before asking the human anything: user-controlled vaults, prior messages, connected files, CRM records, prior Coverage Cat sessions, email threads, document drives, and OCR'd policy documents.
- On a cold start, do not open by asking for name, email, phone, full address, birthday, marital status, driver's license number, or net worth. Read the machine-readable docs, call consumer prefill first with the fullest estimate you can justify, then use Coverage Cat's staged review turn to collect the shopper-owned bundle once.
- Do not re-ask fields already present in `known_summary` unless the user wants to change them.
- Do not surface `needs_more_info` as a step-by-step questionnaire unless your product intentionally falls back to one after exhausting operator-side context. The intended umbrella UX is a single review page followed by offers.
- On every pre-submit user-facing turn, say explicitly that the application is not submitted yet and Coverage Cat has not received a submitted application yet.
- When you list gathered details, estimated answers, or remaining items for the shopper, use short labeled bullets or sections rather than a prose paragraph.
- When any shown value is estimated, mark that bullet or value with `*`, include the short note `* = estimated` once above and once below the list, and do not prefix every estimated line with `[Estimated]`.
- If Coverage Cat stages a review plus shopper-detail `next_question`, show the assembled review and ask for the requested shopper-owned bundle together in one message, in plain English rather than field names. Keep the assembled review and the requested items in labeled bullets or sections. When it includes full name, email, phone, full address, birthday, marital status, driver's license state, driver's license number, and net worth, keep that as one turn instead of splitting it up.
- Any value coming from CRM, documents, email threads, or heuristics that the human has not directly confirmed yet should be sent in `intake` with a matching `field_estimates` row carrying `field`, `source`, and `confidence`.
- Do not ask for credit consent until the user has chosen an offer and Coverage Cat requests it at `select`.
- Use `resume_url` only as a browser fallback. When Coverage Cat reaches `payment_needed` or carrier-managed signing, hand the user to the returned browser URL instead of collecting payment or e-sign details in chat.
- In sandbox mode, use fake or test contact details and stop at `status`. Do not treat sandbox offers, tokens, or links as a real customer handoff.

## API Loop

### 1. Start or update the delegated session

Call `POST /api/agent/umbrella/draft`.

- Omit `uid` to create a new session.
- Start with the fullest `intake` patch you can build from approved operator-side context, not just the fields the human typed in the current turn.
- Include `field_estimates` for every value you are sending in `intake` that is not already directly user-confirmed. Omit `field_estimates` only for user-confirmed values.
- Send only changed fields on follow-up calls.
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
- Do not ask for credit consent yet. Preliminary quotes do not require it; Coverage Cat collects the soft-credit pull authorization after the user chooses an offer.
- Only proceed once the user has explicitly confirmed the reviewed application is correct.

### 3. Quote step

Call `POST /api/agent/umbrella/quotes` with:

- `uid`
- `review_token`
- `review_confirmed: true`

Rules:

- Preliminary umbrella quotes do not require credit consent. If the application changed, go back to `draft` and wait for a fresh `review_token`.
- If this is a sandbox session, the returned offers are mocked and later live-continuation endpoints are intentionally unavailable.

If the response is `quoted`:

- Lead with `response_summary`.
- Show each offer separately with carrier, annual price, coverage limit, any `minimum_requirements`, and `display_notes`.
- If more than three offers are returned, show the recommended offer first and then the best alternative carrier when one exists before summarizing the remaining offers.
- If `minimum_requirements` is present, render it as a visually distinct block before the general notes.
- Use `selection_guidance` to explain the suggested default choice.
- Make clear that premiums are estimates until final underwriting.
- If the response also includes `sandbox: true`, stop at display/testing: do not call `select`, `bind`, or `attach` on this `uid`.

If the response is `no_quotes`:

- Tell the user Coverage Cat did not return an umbrella offer for this profile right now.
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
- If Coverage Cat returns `credit_consent_required` because consent was not sent or was declined, render `credit_consent_question` for the real user and retry `select` with the same `selection_token` once they answer `Yes`.

Do not call this step for sandbox sessions. Sandbox umbrella currently stops at mocked `quotes` and `status`.

Handle the response like this:

- `credit_consent_required`: ask `credit_consent_question` now and retry `select` with the same `selection_token` plus the user's affirmative consent.
- `needs_more_info_to_bind`: keep the user in chat. Ask only the fields in `next_question`, and also offer `offers_url` so the user can finish that same step inside Coverage Cat's GUI if they prefer. Then call the bind endpoint.
- `documents_needed`: Coverage Cat accepted the selection and now needs the user's current home and auto declarations before it can verify the policy and request payment. Upload them through `attach` if you already have them in context. Otherwise, ask the user to share them, and also offer `offers_url` so they can complete the upload in Coverage Cat's GUI if that is easier.
- `documents_pending_review`: Coverage Cat has the declarations and is reviewing them. Do not ask for payment yet. Poll `status`.
- When `documents_pending_review` includes `poll_after_seconds`, wait that long before polling again unless the user asks for an update sooner.
- `payment_needed`: Coverage Cat finished declarations review and is ready for secure payment setup. Share `payment_url`, then poll `status`.
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
- `documents_pending_review`: tell the user verification is in progress and poll `status`.
- Use `poll_after_seconds` from the API response when present. If the status remains pending, tell the user verification is still in progress. There is no extra customer action unless Coverage Cat asks for updated documents.
- `payment_needed`: share `payment_url` and ask the user to complete secure payment setup there.
- `chosen`: keep the user updated through `status`. Share `offers_url` if they want Coverage Cat's offers page for review.

#### Declarations pages during bind

For Monoline and Markel, proactively collect the user's current auto and home declarations pages once the selected offer reaches `documents_needed`.

- If you have access to the user's email, file storage, or photos, search for these documents first (look for PDFs or images with terms like "declarations", "dec page", "policy summary", or the carrier name).
- If you find candidates, confirm with the user before using them.
- If you cannot find them automatically, ask the user to share or photograph them.
- Forward any documents you obtain via `POST /api/agent/umbrella/attach` so Coverage Cat can review them before payment.
- `attach` is only for Monoline or Markel after an offer has been selected. It is not a generic file-upload endpoint for pre-quote drafts.

### 6. Status step

Use `POST /api/agent/umbrella/status` to keep the user in chat after selection.

For sandbox sessions, use `status` only to rehearse the mocked draft/review/quote loop. If the payload includes `sandbox: true`, keep treating every link, offer, and next step as test data.

Interpret the main statuses like this:

- `documents_needed`: Coverage Cat needs declarations uploads before payment. Upload any files you already have, or ask the user to provide them, and offer `offers_url` so they can complete the upload in Coverage Cat's GUI if they prefer.
- `documents_pending_review`: Coverage Cat is reviewing the uploaded declarations. Tell the user verification is ongoing and do not ask for payment yet.
- If the API returns `poll_after_seconds`, use that as the default poll cadence. If the status stays pending, keep the user informed that verification is still in progress.
- `payment_needed`: Coverage Cat verified the declarations and is ready for payment collection. Share `payment_url`.
- `waiting_on_carrier`: no user action is needed. Tell the user Coverage Cat is waiting on the carrier's follow-up path. Offer `offers_url` if they want to review the selected offer. For agency-pay carriers, this means declarations and payment are already on file. For RLI-style flows, any remaining signing or payment link comes from the carrier.
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
