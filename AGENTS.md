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
- Pick the consumer-prefill path first when no operator bearer key is available.
- On the consumer-prefill path, keep the browser closed for as long as Coverage Cat is still returning structured review, quote, or post-choose follow-up data over the freshest rotated `intake_access_token`.
- Do not mix consumer-prefill and delegated loops in one session.
- Do not invent Coverage Cat endpoints or flow steps; start with the published discovery JSON and OpenAPI.
- Do not use customer emails for operator-key issuance.
- Do not include customer credentials, bearer tokens, OTP codes, or private back-office URLs in repo changes.
- Do not copy internal Coverage Cat-only skills or eval tooling into this public repo.
