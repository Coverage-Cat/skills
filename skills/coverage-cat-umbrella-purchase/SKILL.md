---
name: coverage-cat-umbrella-purchase
description: Use when an operator-authenticated agent needs Coverage Cat to run a delegated umbrella quote, review, offer selection, declarations upload, or bind-continuation workflow. Start with the public discovery JSON at https://www.coveragecat.com/api/agent, the OpenAPI at https://www.coveragecat.com/api/agent/openapi.yaml, and the runtime markdown at https://www.coveragecat.com/api/agent/skill.md.
---

# Coverage Cat Umbrella Purchase

Use this skill only for Coverage Cat's delegated umbrella purchase flow.

## Start here

1. Fetch `https://www.coveragecat.com/api/agent`.
2. Fetch `https://www.coveragecat.com/api/agent/openapi.yaml` or `https://www.coveragecat.com/openapi.json`.
3. Read `https://www.coveragecat.com/api/agent/skill.md`.

## Use when

- The user wants Coverage Cat to run a live umbrella quote workflow.
- An operator already has an issued bearer key.
- The job needs Coverage Cat's review, quote, selection, or bind progression rather than a read-only estimate.

## Do not use when

- The task is homeowners quoting; use the homeowners skill instead.
- The task is just estimation or agent discovery; use the agent-tools skill instead.
- You do not have an operator-issued bearer key for the delegated flow.

## Guardrails

- Reuse approved operator-side context before you ask the user for more information.
- Do not ask for credit consent until Coverage Cat returns the documented review state.
- Poll the published status surface instead of inventing a callback flow.

