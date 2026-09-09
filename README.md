# Coverage Cat Skills

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
- Discovery JSON: `https://www.coveragecat.com/api/agent`
- OpenAPI: `https://www.coveragecat.com/api/agent/openapi.yaml`
- Recommended MCP registration URL: `https://www.coveragecat.com/.well-known/mcp.json`
- Skills hub: `https://www.coveragecat.com/ai/skills`

## Consumer-operated MCP default

- If an agent first lands on `coveragecat.com`, start with `https://www.coveragecat.com/agents.txt` or `https://www.coveragecat.com/.well-known/agents.txt`, then prefer the MCP or REST surfaces those files point to. Do not drive the human browser UI unless Coverage Cat should take over the shopper-facing flow.
- Register `https://www.coveragecat.com/.well-known/mcp.json` in ChatGPT and other manifest-aware hosts when the runtime supports MCP.
- For local Codex direct transport, use `https://www.coveragecat.com/api/consumer/mcp` instead of the manifest, then restart the session and check `/mcp`.
- Stay on the consumer purchase subset: `umbrella_consumer_prefill`, `homeowners_consumer_prefill`, `consumer_intake_issues`, `consumer_intake_patch`, plus `umbrella_consumer_select`, `umbrella_consumer_bind`, and `umbrella_consumer_attach` for umbrella post-quote follow-up.
- The umbrella consumer subset can stay on the no-operator path through `payment_needed` and `payment_url`; do not switch to delegated tools just to reach checkout.
- Homeowners consumer agents should stay on the no-operator subset through review and quote follow-up, then let Coverage Cat's consumer portal finish final bind.
- Use `https://www.coveragecat.com/api/agent/mcp` only for delegated operator workflows that already have bearer auth or OAuth support.
- If local Codex delegated OAuth fails because the host rejects the local callback URL, keep shopper flows on the consumer MCP and use the delegated operator MCP only from a bearer-token setup or an OAuth host whose callback policy is already compatible.

## Source of truth

- `docs/agents/`: canonical agent-facing OpenAPI and runtime markdown sources that Coverage Cat syncs into the private app repo at build time.
- `skills/`: installable public skill packages for agent runtimes and the Skills CLI.
