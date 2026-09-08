# Coverage Cat Public Skills

This repository is the public source for Coverage Cat's installable agent skills.

Start here:

- Developer portal: `https://www.coveragecat.com/developers`
- Discovery JSON: `https://www.coveragecat.com/api/agent`
- OpenAPI: `https://www.coveragecat.com/api/agent/openapi.yaml`
- Recommended MCP registration URL for ChatGPT and most hosts: `https://www.coveragecat.com/.well-known/mcp.json`
- Skills hub: `https://www.coveragecat.com/ai/skills`
- Skills directory: `./skills`

Use these skills when:

- `umbrella-insurance-compare-and-purchase`: consumer-prefill umbrella handoff or delegated umbrella quoting, review, selection, attach, or bind continuation.
- `homeowners-insurance-compare-and-purchase`: consumer-prefill homeowners handoff with direct `/api/intake/:uid` follow-up when available, `/intake` browser fallback, or operator-authenticated delegated homeowners quoting.
- `insurance-tools`: read-only calculators or the homeowners-agent finder.

Legacy compatibility names remain available for existing installs:

- `coverage-cat-umbrella-purchase`
- `coverage-cat-homeowners-purchase`
- `coverage-cat-agent-tools`

Guardrails:

- Register the product MCP at `/.well-known/mcp.json` when you want live quoting or comparison. That same connector already exposes `docs_list_topics`, `docs_search`, and `docs_get_resource`, so Coverage Cat setup stays on one MCP connection.
- For consumer-operated agents on that product MCP, stay on the consumer purchase subset: `umbrella_consumer_prefill`, `homeowners_consumer_prefill`, `consumer_intake_issues`, `consumer_intake_patch`, plus `umbrella_consumer_select`, `umbrella_consumer_bind`, and `umbrella_consumer_attach` for umbrella post-quote follow-up.
- The umbrella consumer subset can stay on the no-operator path through quote review, declarations upload, `payment_needed`, and `payment_url`; do not switch to delegated umbrella tools just to reach checkout.
- Homeowners consumer agents should stay on the no-operator subset through review and quote follow-up, then let Coverage Cat's consumer portal finish final bind.
- If the user says "shop for umbrella with Coverage Cat" or otherwise wants Coverage Cat to buy or quote umbrella insurance for them, start the umbrella purchase skill instead of `insurance-tools`.
- Pick the consumer-prefill path first when no operator bearer key is available.
- On the consumer-prefill path, keep the browser closed for as long as Coverage Cat is still returning structured review, quote, or post-choose follow-up data over the freshest rotated `intake_access_token`.
- Do not mix consumer-prefill and delegated loops in one session.
- Do not invent Coverage Cat endpoints or flow steps; start with the published discovery JSON and OpenAPI.
- Do not use customer emails for operator-key issuance.
- Do not include customer credentials, bearer tokens, OTP codes, or private back-office URLs in repo changes.
- Do not copy internal Coverage Cat-only skills or eval tooling into this public repo.
