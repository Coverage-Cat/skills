---
name: coverage-cat-homeowners-purchase
description: "Use for Coverage Cat homeowners quoting. This skill covers two paths: a consumer-prefill handoff that assembles one review-and-consent page from the user's own context, and an operator-partner delegated loop that fills the application from approved CRM, loan, prior-session, email, or policy context, gets one review plus soft-credit consent, submits, then checks for quotes."
---

# Coverage Cat Homeowners Purchase Skill

You are helping a user get homeowners quotes. This skill covers both the consumer-prefill handoff and the operator-partner delegated homeowners loop. Use the separate umbrella purchase skill for umbrella quoting.

## Machine-Readable Contract

Read the discovery and schema surfaces before you start guessing endpoint names or payloads:

1. `GET /api/agent` lists the umbrella and homeowners operation maps, including the delegated homeowners fix-issues email endpoint and the direct-intake follow-up URL templates.
2. `GET /api/agent/openapi.yaml` is the authoritative request/response schema for the published homeowners and consumer handoff endpoints.
3. The direct-homeowner follow-up API lives under `/api/intake/:uid/...` and is described in this skill even though it is not part of the delegated operator loop.

## API Contract Quick Reference

Pin the stable wire contract with `Coverage-Cat-API-Version: v1` when you want a fully explicit request shape, and send `Idempotency-Key: ...` on write calls you may retry. Coverage Cat also echoes `X-Skill-Version: 1.0.0` so you can log which published runtime-contract revision served the response. Example consumer-prefill write:

```bash
curl -X POST https://www.coveragecat.com/api/consumer/homeowners/prefill \
  -H "Content-Type: application/json" \
  -H "Coverage-Cat-API-Version: v1" \
  -H "Idempotency-Key: consumer-home-prefill-001" \
  -d '{"credit_consent_pending":true,"intake":{"full_name":"Taylor Example","home":[{}]}}'
```

Sandbox and errors: the consumer-prefill handoff does not use a separate sandbox flag, while the delegated operator path uses top-level `sandbox: true` only on the first create call for a rehearsal `uid`. The common workflow-facing `error` values are `invalid_request | not_found | conflict | rate_limited | idempotency_conflict | stale_token | sandbox_unsupported | credit_consent_required | internal_error`; keep `message` for display or logging context.

Payment progression: delegated homeowners quoting does not expose a separate payment API step today. Once offers are ready, Coverage Cat hands the homeowner to its secure portal for final selection, confirmation of estimated answers, payment, and bind.

## Choose the Path

This skill supports two different jobs. Pick one path first, because the consumer-prefill path and the operator-partner path do not share the same loop.

1. Use the consumer-prefill path when the homeowner's own AI agent can gather facts from their vault, prior messages, or connected files before handing them to Coverage Cat.
2. Use the operator-partner path when you have a real Coverage Cat operator key and approved back-office context you can use to prefill the application.
3. Do not mix the two paths in one session. Path 1 is an unauthenticated handoff into Coverage Cat's own review page and then its normal `/intake` and portal flow. Path 2 uses the delegated API and dashboard.

## Golden Paths

### Path 1: Consumer prefill handoff

1. Use this path when the shopper wants their own AI agent to gather the fullest homeowners application from user-controlled context before a single review step, or when you do not have an operator key.
2. Call `POST /api/consumer/homeowners/prefill` with the fullest `intake` you can assemble, any matching `field_estimates`, and `credit_consent_pending: true`.
3. Show one review card from `review_summary`, with the soft-credit consent prompt first and the prefilled application below it.
4. Open `resume_url` so Coverage Cat can render the same one-shot review-and-consent page on `/intake?resume=...`.
5. Optionally poll `GET /api/consumer/status?token=...` with the returned `polling_token` for coarse non-PII progress and quote highlights after the handoff.
6. If your runtime cannot prefill, fall back to the current environment origin plus `/intake`. In production that is `https://www.coveragecat.com/intake`.
7. If you need to rehearse the plain browser fallback, send the tester to the current environment origin plus `/intake?ltp=owned_homes&sandbox=homeowners`, use fake or test contact details, and expect mocked homeowners offers on the normal consumer offers page after submit. This browser sandbox does not need an operator key and does not send live customer email or carrier traffic.

#### Optional first-party follow-up API for Path 1

Use this only when a direct-homeowner assistant already has approved access to Coverage Cat's intake bearer token and a real intake `uid`.

1. Call `GET /api/intake/:uid/issues` to inspect what still blocks submission, the returned schema hints, any estimated answers waiting for review, and the GUI handoff links.
2. Read the returned payload under `resource`. The important fields are `resource.status`, `resource.missing_fields`, `resource.review_estimates`, `resource.fix_issues_request_login_url`, and the sensitive direct session link `resource.fix_issues_portal_url`.
3. Use `PATCH /api/intake/:uid` to write user-confirmed answers, or to write estimated homeowners values plus matching `field_estimates` rows when your own context can defensibly fill them.
4. Keep the human out of the loop until `resource.status` becomes `ready_for_review`. At that point, render one review step or hand the user to `resource.fix_issues_request_login_url` so Coverage Cat's GUI can handle the same review.
5. The intended direct-assistant UX is that the only user interruption is that one-time review of estimated answers. If you do not have this API access, fall back to the consumer-prefill handoff or the plain browser handoff at `/intake`.

### Path 2: Operator-partner delegated flow

1. Start with the fullest intake you can build from approved operator-side context.
2. Reuse one `uid` through the whole lifecycle, and keep filling `missing_fields` from your own systems first.
3. If you are rehearsing the flow, set top-level `sandbox: true` on the very first create call only. That `uid` stays sandbox-scoped, the first fully complete submit with explicit credit consent returns `pending_quotes`, and a later poll returns mocked offers without live customer email or carrier traffic.
4. Hold back `credit_check_authorized` until the real homeowner reviews the assembled application and says yes to the soft-credit prompt.
5. After the first fully complete submit with explicit credit consent, expect `pending_quotes`, then poll the same `uid` or use the operator dashboard APIs.
6. If the delegated intake still needs fixes, use `homeowner_fix_issues_request_login_url` or `POST /api/agent/homeowners/fix-issues-email` to hand the homeowner back to Coverage Cat's GUI without exposing a direct session URL.
7. When offers are ready, summarize them and hand the homeowner to Coverage Cat with the safe sign-in request link for final selection and bind.

## Authentication

- The operator must give you a real Coverage Cat operator API key ahead of time.
- Send that key as `Authorization: Bearer <key>`.
- Do not call Coverage Cat's OTP/key-issuance endpoints from inside the agent for this flow.
- Shared environment keys are not accepted for the homeowners purchase endpoint.
- Repeated invalid bearer-key attempts may return `429 too_many_attempts`. Honor the `Retry-After` header before retrying.

## Goal

1. Search approved operator-side context first and assemble the fullest homeowners application you can before involving the human.
2. On the consumer-prefill path, do the same with the user's own vault, prior messages, and connected files before you ask a single question.
3. Send non-user-confirmed home values in `intake.home[]` with matching `field_estimates[]` rows so Coverage Cat can preserve provenance and mark them for later review.
4. Keep `needs_more_info` behind the scenes when you can. The intended UX is a single review-and-consent handoff, not a long questionnaire.
5. Withhold `credit_check_authorized` until the real homeowner has reviewed the assembled application and explicitly said yes to a soft credit pull.
6. After the successful initial submit, expect `pending_quotes` and keep the quote wait asynchronous by polling the same `uid` or using the homeowners dashboard APIs.
7. Once offers are ready, summarize them clearly and hand the homeowner to Coverage Cat's secure portal for final selection, estimated-field confirmation, and bind.
8. If any response includes `sandbox: true`, treat every offer, status, and link as mocked test data and do not forward it to a real homeowner.

## Conversation Rules

- Lead with a compact framing statement such as: "I can gather your homeowners application, confirm it with you once, and then keep checking for quotes."
- Pick the path first. Use the consumer-prefill handoff when no operator bearer key is present, and use the delegated operator loop only when a real key is already available.
- Reuse known facts and search the available context before asking the human anything: user-controlled vaults, prior messages, connected files, CRM records, loan files, prior Coverage Cat sessions, email threads, document drives, and OCR'd policy documents.
- Do not re-ask fields already present in `known_summary` unless the user wants to change them.
- Do not set `credit_check_authorized` to `true` until the real homeowner has reviewed the assembled application and explicitly answered yes to the soft-credit prompt.
- If Coverage Cat is only missing `credit_check_authorized`, render a single review step: put the soft-credit explanation and consent prompt first, then show the current application summary or JSON below it.
- If the user corrects anything during that review, patch the same `uid` first, then rerun the same one-time review step before resubmitting.
- In sandbox mode, use fake or test contact details and never treat the returned offers or links as a real customer handoff.
- Do not use this homeowners workflow for umbrella purchase. Use the dedicated umbrella purchase skill instead.

## Endpoint

Call `POST /api/agent/homeowners/quotes`.

This endpoint is only for Path 2, the operator-partner delegated loop. Do not use it for the consumer-prefill handoff.

This is the primary endpoint for the full delegated homeowners loop:

1. Create a new homeowners intake by omitting `uid`.
2. Update the same intake by sending the returned `uid`.
3. When the only remaining gap is `credit_check_authorized`, keep the same `uid`, show the single review step, and only resend with `credit_check_authorized: true` after the real homeowner explicitly says yes.
4. After that successful submit, expect `pending_quotes`.
5. Poll for quote progress by sending the same `uid` again until `pending_quotes` becomes `quoted`.

### Sandbox

Set top-level `sandbox: true` only on the first create call when you want a mocked delegated-homeowners rehearsal.

- The sandbox flag sticks to that `uid`, so later calls may omit it.
- Sandbox reuses the real validation and one-time review/credit-consent checkpoint.
- Sandbox does not send live customer emails or carrier traffic.
- The first successful sandbox submit returns `pending_quotes`.
- A later poll on that same `uid` returns mocked homeowners offers with `sandbox: true`.
- Do not forward sandbox offers or links to a real homeowner.

You may also call `GET /api/agent/homeowners/dashboard` after any write to retrieve non-PII property status rows for the same operator key, or `POST /api/agent/homeowners/dashboard/session` to mint a short-lived browser dashboard link for that operator. Use those dashboard endpoints when you need to check the status of outstanding delegated homeowners requests across the operator rather than repeatedly polling one intake.

If a delegated intake is still blocked on missing data and you want Coverage Cat to email the homeowner a secure sign-in link to the fix-issues flow, call `POST /api/agent/homeowners/fix-issues-email` with `{"uid":"..."}`. The response also includes `homeowner_fix_issues_request_login_url` for safe chat handoff.

## Request Shape

Start with the fullest `intake` payload you can assemble from approved operator-side context, not just the fields the human typed in the current turn.

- Always reuse the same `uid` after the first response.
- Send directly user-confirmed applicant and property facts under `intake`.
- When you send a home value that is not directly user-confirmed but is defensibly backed by CRM, a loan file, email threads, prior Coverage Cat sessions, or policy documents, still send that field in `intake.home[]` and add a matching `intake.home[].field_estimates[]` row for it.
- Use the same `field_estimates` object shape Coverage Cat returns in responses. Set `confirmed_by_user: false` until a human reconfirms that value.
- Put home details under `intake.home` as an array, even for one home.
- Include both applicant-level facts and property-level facts when available.
- Prefer one large, accurate payload over many tiny incremental payloads.
- Resend the latest known facts when your system has better data. Coverage Cat will merge them.
- Omit `credit_check_authorized` or leave it `false` while you are still enriching from operator-side context. Only send `credit_check_authorized: true` after the real homeowner has reviewed the assembled application and explicitly authorized a soft credit pull that does not affect their credit score.
- If the home has a mortgage and your CRM or loan file has any of them, send optional `intake.home[].loans[]` fields such as `lender_name`, `loan_number`, `mortgagee_clause`, and `loan_effective_date`. Coverage Cat accepts partial loan rows, so you may send only the fields you have.

Example first call when your systems already carry a defensible but not yet user-confirmed home value:

```json
{
  "intake": {
    "full_name": "Taylor Home",
    "email": "taylor.home@example.com",
    "age": "1988-04-01",
    "marital_status": "single",
    "address": {
      "streetnumber": "4225",
      "street": "Emory Ave",
      "city": "West University Place",
      "state": "TX",
      "zip": "77005"
    },
    "home": [
      {
        "address": {
          "streetnumber": "4225",
          "street": "Emory Ave",
          "city": "West University Place",
          "state": "TX",
          "zip": "77005"
        },
        "home_ownership": "owned_primary",
        "property_type": "single_family",
        "new_purchase": false,
        "field_estimates": [
          {
            "field": "property_type",
            "value": "single_family",
            "source": "crm",
            "confidence": "medium",
            "confirmed_by_user": false,
            "estimated_at": "2026-08-11T19:12:00Z"
          }
        ]
      }
    ]
  }
}
```

Those `field_estimates` rows are what tell Coverage Cat to keep the value moving through delegated quoting while still requiring the homeowner to review or correct it before final bind in the portal.

Example first sandbox rehearsal call:

```json
{
  "sandbox": true,
  "intake": {
    "full_name": "Taylor Sandbox",
    "email": "sandbox-homeowner@example.test",
    "age": "1988-04-01",
    "marital_status": "single",
    "address": {
      "streetnumber": "4225",
      "street": "Emory Ave",
      "city": "West University Place",
      "state": "TX",
      "zip": "77005"
    },
    "home": [
      {
        "address": {
          "streetnumber": "4225",
          "street": "Emory Ave",
          "city": "West University Place",
          "state": "TX",
          "zip": "77005"
        },
        "home_ownership": "owned_primary",
        "property_type": "single_family",
        "new_purchase": false
      }
    ]
  }
}
```

### Address format

Send addresses as structured objects, not as a single free-form string.

- Put the applicant's primary residence at `intake.address`.
- Put each insured property's address at `intake.home[].address`.
- For a typical owner-occupied single-home case, send the same address in both places.
- Coverage Cat does not require Google-normalized or USPS-standardized formatting, but the fields must already be split sensibly.
- `state` may be a two-letter code or a full state name. Coverage Cat normalizes it to the two-letter code.
- `zip` must be a 5-digit ZIP code.

Recommended shape:

```json
{
  "streetnumber": "4225",
  "street": "Emory Ave",
  "city": "West University Place",
  "state": "TX",
  "zip": "77005"
}
```

Do not send this instead:

```json
{
  "street": "4225 Emory Ave, West University Place, TX 77005"
}
```

That free-form version may persist as low-quality data and still leave the intake missing other required homeowners fields.

### Property IDs

Coverage Cat returns a non-PII `property_id` for each home in the response summaries, and for quoted offers when an offer can be matched to a property address.

- Use `property_id` when you need to track a property in your own logs or dashboards without relying on client names.
- `property_id` is intended for operator-side correlation, not for customer-facing display.
- `property_id` is stable for the same operator plus normalized property address. A new `uid` for the same address will usually reuse the same `property_id`. Treat it as a property correlation key, not an intake-row ID.
- If you need the latest status for one property after a quote call, query the homeowners dashboard API and filter by either `uid` or `property_id`.

Useful fields commonly needed for homeowners quoting include:

- `full_name`
- `email`
- `age` (sent as an ISO date of birth) or the accepted aliases `birthday` / `date_of_birth`
- `marital_status`
- `address`
- `credit_check_authorized`
- `sms_consent`
- `home`

### Home ownership values

Send `intake.home[].home_ownership` as one of the canonical values:

- `owned_primary`: owner-occupied primary residence
- `owned_seasonal`: owner-occupied secondary or vacation home
- `owned_rental`: owned only to rent to others
- `rental`: the applicant rents from someone else
- `under_construction`: owned but not yet complete
- `informal_basis`: lived in informally, such as a parent's home

Two shorthand aliases are also accepted for convenience:

- `owned`: normalized to `owned_primary`
- `new_purchase`: normalized to `owned_primary` and sets `new_purchase: true`

If you want to opt in to a non-primary ownership shape, send the canonical value directly.

### Looking-to-protect

This endpoint always scopes the intake to homeowners (`looking_to_protect: ["owned_homes"]`). Any `looking_to_protect` you send is ignored. Use the umbrella or general delegated flows for other lines.

## Response Handling

### `needs_more_info`

Coverage Cat still needs either more application data or the real homeowner's final credit authorization before it can submit.

- If the response also includes `sandbox: true`, treat this as a mocked rehearsal state. Do not forward the returned links or status text to a real homeowner.
- If `missing_fields` is only `credit_check_authorized`, treat this as the one-time review checkpoint rather than a normal missing-data failure.
- In that credit-consent checkpoint, explain that Coverage Cat needs the real homeowner's explicit authorization for a soft credit pull that does not affect their credit score.
- Render a single review step: put that consent prompt first, then show the current application summary or JSON below it.
- Do not set `credit_check_authorized` to `true` until the real homeowner personally says yes.
- If the homeowner corrects anything during review, patch the same `uid` first, then rerun the same one-time review step before asking for consent again.
- After explicit yes, call `POST /api/agent/homeowners/quotes` again with the same `uid` and `intake.credit_check_authorized: true`.
- Read `missing_fields`.
- Treat this as an operator-side enrichment loop, not the default customer questionnaire.
- Fill as many of those fields as you can from the operator's CRM, loan file, email threads, prior Coverage Cat sessions, or policy documents.
- Any new non-user-confirmed home values you send on the retry should also carry matching `intake.home[].field_estimates[]` rows.
- Retry `POST /api/agent/homeowners/quotes` with the same `uid`.
- Do not start a second intake unless Coverage Cat explicitly tells you the old one is unusable.
- Expect the list to narrow as Coverage Cat validates state- and property-specific requirements. Keep following the latest `missing_fields` list for that same `uid`.
- If you correct a previously bad address or any other field, patch the same `uid` with the corrected data. Do not abandon the intake just because an earlier payload was malformed.
- If Coverage Cat still responds with `needs_more_info` after an address correction, treat that as "this same intake still needs other fields" unless the response explicitly says the update could not be applied.
- If Coverage Cat returns `homeowner_fix_issues_request_login_url`, offer that safe sign-in page whenever you want the homeowner to complete the remaining items in Coverage Cat's GUI instead of continuing in chat.
- If you want Coverage Cat itself to send the homeowner that same handoff by email, call `POST /api/agent/homeowners/fix-issues-email` with the same `uid`.
- Ask the user only for the remaining fields after approved operator-side context is exhausted, or hand off if your workflow prefers a human follow-up.

### `pending_quotes`

Coverage Cat has a complete submitted homeowners intake and is waiting for quotes.

- This is the expected immediate status after the successful review-and-consent-backed submit.
- Keep the returned `uid`.
- Poll the same endpoint with that `uid`. A one-minute cadence is a good default when the user is waiting live.
- `homeowner_quotes_request_login_url` is the safer homeowner handoff link. It lands on Coverage Cat's email-first sign-in page for the offers view, so it does not hand over a direct session link. Default to sharing it at `quoted`. If you want to pre-stage sign-in, you may share it during `pending_quotes`.
- `homeowner_portal_url` is the direct Coverage Cat consumer portal session link for this intake. Treat it as sensitive session data rather than the default handoff URL.
- `offers_page_url` is the same portal link, kept for backwards compatibility.
- `operator_dashboard_url` is a short-lived browser login link for the operator-side homeowners dashboard, already filtered to this `uid`.
- If you need a status board for many outstanding delegated homeowners requests, use `GET /api/agent/homeowners/dashboard` or mint a fresh browser session with `POST /api/agent/homeowners/dashboard/session`.
- If the response includes `sandbox: true`, the next poll returns mocked offers. No live quote jobs or customer emails were started.

### `field_estimates`

Homeowners applications often need answers that a mortgage loan file does not carry. When Coverage Cat can derive a defensible answer from a property probe or an internal estimator, it fills the answer and reports it as an estimate. The intake keeps moving instead of stopping on unknowns.

Each row on `known_summary.homes[].field_estimates` reflects an estimated home value that either the operator submitted or Coverage Cat filled from its own enrichment. When you send estimated home values yourself, add them on `intake.home[].field_estimates[]` using the same shape:

```json
{
  "field": "property_type",
  "value": "single_family",
  "source": "zillow",
  "confidence": "high",
  "confirmed_by_user": false,
  "estimated_at": "2026-08-09T18:32:11Z"
}
```

Rules for handling estimates:

- Estimates can keep quoting moving, but homeowners finalization still happens in Coverage Cat's normal consumer portal. Before the policy is finalized there, Coverage Cat will route the customer back through home fix-issues/review to confirm or correct any unconfirmed estimated home fields.
- If you are sending a defensible but not yet user-confirmed home value, include the actual field value in `intake.home[]` and add a matching `field_estimates` row on that same home. Those rows are what make Coverage Cat treat the value as estimated and send it back through portal review before final bind.
- If the operator's system carries a real (user-confirmed) value for a field Coverage Cat estimated, resend that value in the next `POST /api/agent/homeowners/quotes` payload. Coverage Cat will overwrite the estimate with the confirmed answer.
- `confirmed_by_user` flips once a human resubmits the same answer back through Coverage Cat, whether that happens from the operator payload or the homeowner reviewing the pre-filled value in the portal.
- Common `source` values you may see or send today include operator-side labels such as `crm`, `email_thread`, `policy_declarations`, and `prior_coverage_cat_session`, plus Coverage Cat-generated labels such as `loan_file`, `zillow`, `realtor`, `vintage_rule`, and `zdr_openai`. Treat unknown values as opaque.
- `confidence` is `low | medium | high` and reflects Coverage Cat's internal certainty, not carrier acceptance risk. Prioritize confirming `low` confidence estimates with the customer before they finalize in the portal.

Coverage Cat may estimate any reviewable `intake.home[]` field it can defensibly infer from the property probe, the current intake payload, and the ZDR-backed estimator pass. In practice, the delegated flow can pre-fill the normal homeowners review and editor fields. It is not limited to a small fixed subset, and it still marks every guessed answer in `field_estimates`.

Probe/vintage estimates are still used where they are stronger than an LLM guess:

- `property_type`
- `has_attached_garage`
- `half_bathroom_count`
- `purchase_date`
- `hurricane_resistant_windows` for Florida homes built in or after 2002

Everything else Coverage Cat estimates is surfaced with `source: "zdr_openai"` and must be treated as a guess pending human confirmation in the portal review step.

For enum/list fields, the static OpenAPI document is intentionally permissive. The authoritative vocabulary for a given intake response is the returned `schema` and each `missing_fields[]` item's `enum` list.

### Loan-file gaps that are NOT estimated

Some homeowners inputs must come from the operator's system, not from Coverage Cat guessing:

- The homeowners **replacement cost** (Coverage A / dwelling coverage) must come from a Replacement Cost Estimator, never from a mortgage appraised value or the property's purchase price. Coverage Cat will not infer dwelling replacement cost from sale price. If your system does not carry a real dwelling replacement-cost figure, leave it blank and let Coverage Cat's carrier-side estimator fill it. Do not reuse personal-property fields such as `valuables_replacement_cost` for dwelling replacement cost, and do not paper over a missing RCE with a sale-price fallback.
- Loan-level facts like `note_rate` and `term_months` are properties of the selected mortgage product, not the loan file. Coverage Cat does not carry them and does not need them for a homeowners quote. Keep them on your side.
- Homeowners premium is priced by Coverage Cat, not the lender. Whether your file escrows insurance is a lender decision that has no effect on the intake. Do not send escrow flags to Coverage Cat, and do not condition binding on escrow status.

### `quoted`

Coverage Cat has home offers ready.

- Lead with the summarized `offers`.
- Mention `recommended: true` offers first.
- Share `homeowner_quotes_request_login_url` when you need a pass-through link for the customer. That sends them to Coverage Cat's email-first sign-in page before they reach their offers view.
- `homeowner_portal_url` is still returned as the direct secure portal session link for this intake, but it should be handled as sensitive session data rather than the default share link.
- `offers_page_url` is the same portal link, kept for backwards compatibility.
- Use `operator_dashboard_url` when a human operator needs Coverage Cat's browser view of the same delegated intake.
- Treat secure URLs as sensitive session data.
- If the response includes `sandbox: true`, the offers and links are mocked and should only be used to test your quote-display and customer-handoff logic.

### Property Status Dashboard API

Call `GET /api/agent/homeowners/dashboard` with the same operator bearer key to read non-PII property status rows after a delegated homeowners call. This is the operator-side status view for outstanding delegated homeowners requests.

Useful query parameters:

- `uid`: limit results to a single delegated homeowners request
- `property_id`: limit results to a single property
- `status`: one of `needs_more_info`, `pending_quotes`, `quoted`, or `quotes_published`
- `limit`: maximum number of rows to return

Each row is keyed by `property_id` and includes a coarse status, `offers_published` as a simple yes/no flag, location summary (`city`, `state`, `zip` only), property facts, safe homeowner sign-in request links, and the originating request `uid`.

Do not expect this dashboard surface to diagnose exact missing fields or show exact offer counts. For missing-data details, use the live `missing_fields` returned by the delegated quote response for that `uid`, or the direct-homeowner issues endpoint when you are operating in the first-party direct flow.

If you need a browser session instead of raw JSON, call `POST /api/agent/homeowners/dashboard/session` with the same optional filters (`uid`, `property_id`, `status`, `limit`). Coverage Cat returns a short-lived `dashboard_url` that signs the operator into the browser dashboard without exposing the bearer key in the URL.

When a row includes `homeowner_quotes_request_login_url`, you may pass that link to the homeowner. It sends them to Coverage Cat's email-first sign-in page for the offers view, so it does not log the operator or agent into the homeowner's private offers page.

When a `needs_more_info` row includes `homeowner_fix_issues_request_login_url`, you may pass that link to the homeowner so they land on Coverage Cat's fix-issues GUI rather than the offers page. If `fix_issues_email_available` is true, the operator dashboard and `POST /api/agent/homeowners/fix-issues-email` may also trigger Coverage Cat to send that handoff by email.

## Operational Rules

- Start with the fullest approved-context payload you can assemble before asking the human for more data.
- Send directly user-confirmed values normally, and send non-user-confirmed but defensible home values in `intake.home[]` with matching `field_estimates` metadata.
- Do not fabricate unsupported guesses. If you cannot defend a value from approved operator-side context or Coverage Cat's own enrichment, leave it blank and keep searching or ask the user.
- Reuse the same `uid` until the flow is complete.
- If Coverage Cat says data is missing, fill it from existing systems first and only ask the user when your systems genuinely do not have it.
- When patching an existing intake, prefer resending the current best-known `intake.address` and `intake.home[].address` objects rather than assuming Coverage Cat will infer one from the other.
- Never infer homeowners replacement cost from purchase price or appraised value. If the operator's system does not carry a Replacement Cost Estimator output, leave the field for Coverage Cat's carrier-side estimator to fill.
- The delegated homeowners API stops at intake submission and quoting. The homeowner must choose the offer, review estimated answers, and complete final bind in Coverage Cat's normal consumer portal or the emails Coverage Cat sends.
- Customers should use Coverage Cat's normal offers and intake flow, plus the emails Coverage Cat sends during quoting. The delegated homeowners API does not expose a separate customer status dashboard.
