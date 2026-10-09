# Coverage Cat Public Skills

This repository is the public source for Coverage Cat's installable agent skills.

Start here:

- Public AGENTS.md: `https://www.coveragecat.com/AGENTS.md`
- Public agents.txt: `https://www.coveragecat.com/agents.txt`
- Well-known agents.txt: `https://www.coveragecat.com/.well-known/agents.txt`
- Developer portal: `https://www.coveragecat.com/developers`
- Consumer discovery JSON: `https://www.coveragecat.com/api/consumer`
- Consumer OpenAPI: `https://www.coveragecat.com/api/consumer/openapi.yaml`
- Delegated/tools discovery JSON: `https://www.coveragecat.com/api/agent`
- Delegated/tools OpenAPI: `https://www.coveragecat.com/api/agent/openapi.yaml`
- Recommended MCP registration URL for ChatGPT and most hosts: `https://www.coveragecat.com/.well-known/mcp.json`
- Skills hub: `https://www.coveragecat.com/ai/skills`
- Skills directory: `./skills`

Use these skills when:

- `umbrella-insurance-compare-and-purchase`: the shopper wants Coverage Cat to shop for, quote, compare, or buy umbrella insurance for them; use the consumer-prefill umbrella handoff or delegated umbrella quoting, review, selection, attach, or bind continuation.
- `homeowners-insurance-compare-and-purchase`: consumer-prefill homeowners handoff with direct `/api/intake/:uid` follow-up when available, `/intake` browser fallback, or operator-authenticated delegated homeowners quoting.
- `insurance-tools`: research-only calculators or the homeowners-agent finder, not live shopping or purchase.

Legacy compatibility names remain available for existing installs:

- `coverage-cat-umbrella-purchase`
- `coverage-cat-homeowners-purchase`
- `coverage-cat-agent-tools`

Guardrails:

- Agents landing on `coveragecat.com` should start with `https://www.coveragecat.com/AGENTS.md`, `https://www.coveragecat.com/agents.txt`, or `https://www.coveragecat.com/.well-known/agents.txt`, then prefer the MCP or REST surfaces those files point to. Do not drive the human browser UI unless Coverage Cat should take over the shopper-facing flow.
- Register the default consumer MCP at `/.well-known/mcp.json` when you want live quoting or comparison in ChatGPT or another manifest-aware MCP host. For local Codex direct transport, use `https://www.coveragecat.com/api/consumer/mcp` instead of the manifest, then restart the session and check `/mcp`. That same consumer connector already exposes `docs_list_topics`, `docs_search`, and `docs_get_resource`, so Coverage Cat setup stays on one MCP connection.
- If Coverage Cat MCP tools are unavailable in a runtime that supports MCP, stop and ask a short choice question instead of telling the user to register `/.well-known/mcp.json` immediately.
  Ask the question on its own line exactly as:

  Would you like to install the Coverage Cat MCP (recommended) or use the REST/CLI tools instead?

  Only share `/.well-known/mcp.json` after the user chooses MCP or asks for setup details.
- For consumer-operated agents on that default consumer MCP, stay on the consumer purchase subset: `umbrella_consumer_prefill`, `homeowners_consumer_prefill`, `consumer_intake_issues`, `consumer_intake_patch`, plus `umbrella_consumer_select`, `umbrella_consumer_bind`, `umbrella_consumer_attach`, `umbrella_consumer_payment`, `umbrella_consumer_policy`, `umbrella_consumer_service`, `homeowners_consumer_select`, `homeowners_consumer_bind`, `homeowners_consumer_policy`, and `homeowners_consumer_service`.
- The umbrella consumer subset can stay on the no-operator path through quote review, declarations upload, `payment_needed`, secure payment UI when available, `payment_url` fallback, bound-policy retrieval, and shopper-approved policy-support previews or requests. Do not switch to delegated umbrella tools just to reach checkout or aftercare.
- Use those shopper surfaces only for insurance purchase and policy support. Coverage Cat records connector origin and permitted purpose on these handoffs, and connector data must stay out of unrelated advertising or model-training loops.
- Homeowners consumer agents should stay on the no-operator subset through review, long-lag quote polling, `homeowners_consumer_select`, `homeowners_consumer_bind`, and later policy/support follow-up. When `consumer_intake_issues` returns `conversation.reply_pending`, answer that insurer or shopper follow-up in the same loop with `consumer_intake_patch` and `message_reply`. Coverage Cat does not collect first-party homeowners payment in chat; carrier payment or signing links go directly to the customer's email on file, and quotes or carrier bind follow-up may take up to 48 hours.
- Use `https://www.coveragecat.com/api/consumer` and `https://www.coveragecat.com/api/consumer/openapi.yaml` for personal shopper flows. Use `/api/agent` and `/api/agent/openapi.yaml` only when delegated operator auth or read-only tool APIs are intentionally in scope.
- `GET` on `https://www.coveragecat.com/api/consumer/mcp` returning `405 Method Not Allowed` is expected; direct MCP transport uses `POST` JSON-RPC. Do not use `/.well-known/mcp.json` as the direct transport URL.
- If a local Codex OAuth flow for `https://www.coveragecat.com/api/agent/mcp` fails because the host rejects the local callback URL, keep shopper flows on the consumer MCP and use the delegated operator MCP only from a bearer-token setup or an OAuth host whose callback policy is already compatible.
- If the user says "shop for umbrella with Coverage Cat" or otherwise wants Coverage Cat to buy or quote umbrella insurance for them, start the umbrella purchase skill instead of `insurance-tools`.
- Pick the consumer-prefill path first when no operator bearer key is available.
- On the default consumer MCP, Coverage Cat keeps follow-up authorization in the hidden MCP session after prefill; keep using the same consumer-operated subset with `uid` rather than asking the model to carry an intake bearer token.
- On the shopper REST fallback, keep the browser closed for as long as Coverage Cat is still returning structured review, quote, or post-choose follow-up data over the freshest rotated `intake_access_token`.
- Keep the shopper on the MCP or REST loop until Coverage Cat returns `resume_url` or you intentionally hand the browser over to Coverage Cat. `/umbrella` or `/intake` are fallback browser starts, not the preferred purchase entrypoint.
- Do not mix consumer-prefill and delegated loops in one session.
- Do not invent Coverage Cat endpoints or flow steps; start with the published discovery JSON and OpenAPI.
- Do not use customer emails for operator-key issuance.
- Do not ask a personal shopper for a Coverage Cat operator API key or delegated OAuth just to run consumer-prefill or direct follow-up.
- Do not include customer credentials, bearer tokens, OTP codes, or private back-office URLs in repo changes.
- Do not copy internal Coverage Cat-only skills or eval tooling into this public repo.
