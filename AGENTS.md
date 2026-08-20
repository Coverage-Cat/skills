# Coverage Cat Public Skills

This repository is the public source for Coverage Cat's installable agent skills.

Start here:

- Developer portal: `https://www.coveragecat.com/developers`
- Discovery JSON: `https://www.coveragecat.com/api/agent`
- OpenAPI: `https://www.coveragecat.com/api/agent/openapi.yaml`
- Skills directory: `./skills`

Use these skills when:

- `umbrella-insurance-compare-and-purchase`: delegated umbrella quoting, review, selection, attach, or bind continuation.
- `homeowners-insurance-compare-and-purchase`: direct homeowners handoff or operator-authenticated delegated homeowners quoting.
- `insurance-tools`: read-only calculators or the homeowners-agent finder.

Legacy compatibility names remain available for existing installs:

- `coverage-cat-umbrella-purchase`
- `coverage-cat-homeowners-purchase`
- `coverage-cat-agent-tools`

Guardrails:

- Do not invent Coverage Cat endpoints or flow steps; start with the published discovery JSON and OpenAPI.
- Do not use customer emails for operator-key issuance.
- Do not include customer credentials, bearer tokens, OTP codes, or private back-office URLs in repo changes.
- Do not copy internal Coverage Cat-only skills or eval tooling into this public repo.
