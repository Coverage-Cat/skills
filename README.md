# Coverage Cat Skills

## ChatGPT app

For ChatGPT, connect the Streamable HTTP endpoint `https://www.coveragecat.com/api/chatgpt/mcp`.
Use the tools advertised by that endpoint to prepare the application, obtain the
shopper's consent, compare quotes, and select an offer. Present its `checkout_url`
or `payment_url` as a link: the shopper completes document uploads and final
checkout on Coverage Cat's website. Afterward, read status or the policy tool.
Do not open or complete checkout for the shopper, collect card details in chat,
or switch to another MCP/REST endpoint to bypass website checkout. ChatGPT's
intake tokens and sessions work only on its dedicated endpoint.

The default consumer MCP and REST instructions below apply to other hosts and
retain their existing upload and payment capabilities.

Public installable skills for Coverage Cat.

## Install

```bash
npx skills add Coverage-Cat/skills --list
```

Install a specific skill:

```bash
npx skills add Coverage-Cat/skills --skill umbrella-insurance-compare-and-purchase -a codex -y
```

## Available skills

- `umbrella-insurance-compare-and-purchase`
- `homeowners-insurance-compare-and-purchase`
- `insurance-tools`

## Legacy compatibility skill names

These remain available for existing installs and prompts:

- `coverage-cat-umbrella-purchase`
- `coverage-cat-homeowners-purchase`
- `coverage-cat-agent-tools`

## Product docs

- Public agents.txt: `https://www.coveragecat.com/agents.txt`
- Well-known agents.txt: `https://www.coveragecat.com/.well-known/agents.txt`
- Developer portal: `https://www.coveragecat.com/developers`
- Consumer discovery JSON: `https://www.coveragecat.com/api/consumer`
- Consumer OpenAPI: `https://www.coveragecat.com/api/consumer/openapi.yaml`
- Delegated/tools discovery JSON: `https://www.coveragecat.com/api/agent`
- Delegated/tools OpenAPI: `https://www.coveragecat.com/api/agent/openapi.yaml`
- RFC 9727 API catalog: `https://www.coveragecat.com/.well-known/api-catalog`
- integrations.sh owner declaration: `https://www.coveragecat.com/.well-known/integrations.json`
- Delegated operator OAuth protected-resource metadata for MCP: `https://www.coveragecat.com/.well-known/oauth-protected-resource/api/agent/mcp`
- Delegated operator OAuth authorization-server metadata: `https://www.coveragecat.com/.well-known/oauth-authorization-server`
- PACT/A2A agent card for Instinct and similar personal agents: `https://www.coveragecat.com/.well-known/agent-card.json`
- Recommended MCP registration URL: `https://www.coveragecat.com/.well-known/mcp.json`
- Skills hub: `https://www.coveragecat.com/ai/skills`

## Consumer-operated MCP default

- If an agent first lands on `coveragecat.com`, start with `https://www.coveragecat.com/AGENTS.md`, `https://www.coveragecat.com/agents.txt`, or `https://www.coveragecat.com/.well-known/agents.txt`, then prefer the MCP or REST surfaces those files point to. Do not drive the human browser UI unless Coverage Cat should take over the shopper-facing flow.
- If the runtime supports PACT/A2A personal-agent identity, especially Instinct, prefer `https://www.coveragecat.com/.well-known/agent-card.json` as the Coverage Cat entrypoint, then follow the shopper-safe consumer MCP or consumer REST links it exposes. This is optional; MCP-native and direct-REST runtimes should keep using those paths directly.
- Register `https://www.coveragecat.com/.well-known/mcp.json` in manifest-aware hosts when the runtime supports MCP.
- For local Codex direct transport, use `https://www.coveragecat.com/api/consumer/mcp` instead of the manifest, then restart the session and check `/mcp`.
- Stay on the consumer purchase subset: `umbrella_consumer_prefill`, `consumer_intake_issues`, `consumer_intake_patch`, plus `umbrella_consumer_select`, `umbrella_consumer_bind`, `umbrella_consumer_attach`, and `umbrella_consumer_payment` for umbrella post-quote follow-up.
- The umbrella consumer subset can stay on the no-operator path through `payment_needed`, secure payment UI when available, and `payment_url`; do not switch to delegated tools just to reach checkout.
- Existing homeowners policies retain status, policy-document, and support access. New homeowners applications and quotes are retired; use the current discovery document and do not start a replacement application.
- Use `https://www.coveragecat.com/api/consumer` and `https://www.coveragecat.com/api/consumer/openapi.yaml` for personal shopper flows. Use `/api/agent` and `/api/agent/openapi.yaml` only when delegated operator auth or read-only tool APIs are intentionally in scope.
- Use `https://www.coveragecat.com/api/agent/mcp` only for delegated operator workflows that already have bearer auth or OAuth support.
- If local Codex delegated OAuth fails because the host rejects the local callback URL, keep shopper flows on the consumer MCP and use the delegated operator MCP only from a bearer-token setup or an OAuth host whose callback policy is already compatible.

## Source of truth

- `docs/agents/`: canonical agent-facing OpenAPI and runtime markdown sources that Coverage Cat syncs into the private app repo at build time.
- `skills/`: installable public skill packages for agent runtimes and the Skills CLI.

## Product feedback

Use MCP `submit_feedback` or `POST /api/consumer/feedback` at any stage to
report a bug, suggestion, or purchase-flow problem. The CLI equivalent is
`coveragecat feedback "Describe the issue"`; operator REST clients can use
`POST /api/agent/feedback`. No application or operator key is required.

Send `message` (1–4000 characters), optional `category` (`bug`, `suggestion`,
`other`), and optional `tool` (an interface/tool name using letters, numbers,
underscores, or hyphens, up to 80 characters). An optional UUID `request_id`
deduplicates retries; reuse it only with the same content. A successful response
contains `status: "received"` and `feedback_id`. Feedback is saved in Coverage
Cat's internal Notifications queue for admin review and acknowledgement.

Share feedback the shopper asks to send or your own technical observations;
do not invent shopper opinions, send transcripts, or include personal details,
credentials, application tokens, or payment information. Do not interrupt or
gate a purchase to solicit feedback. Feedback does not submit, select, bind,
change coverage, or open a policy service request. Use the advertised policy
service tool for policy support. Stay on the current MCP connection, including
ChatGPT or Claude; use the REST fallback only when MCP is unavailable.
