# Coverage Cat Public Skills

This repository is the public source for Coverage Cat's installable agent skills.

Start here:

- Developer portal: `https://www.coveragecat.com/developers`
- Discovery JSON: `https://www.coveragecat.com/api/agent`
- OpenAPI: `https://www.coveragecat.com/api/agent/openapi.yaml`
- Skills directory: `./skills`

Use these skills when:

- `coverage-cat-umbrella-purchase`: delegated umbrella quoting, review, selection, attach, or bind continuation.
- `coverage-cat-homeowners-purchase`: direct homeowners handoff or operator-authenticated delegated homeowners quoting.
- `coverage-cat-agent-tools`: read-only calculators or the homeowners-agent finder.

Guardrails:

- Do not invent Coverage Cat endpoints or flow steps; start with the published discovery JSON and OpenAPI.
- Do not use customer emails for operator-key issuance.
- Do not include customer credentials, bearer tokens, OTP codes, or private back-office URLs in repo changes.
- Do not copy internal Coverage Cat-only skills or eval tooling into this public repo.
