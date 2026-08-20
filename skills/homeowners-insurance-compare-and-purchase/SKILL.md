---
name: homeowners-insurance-compare-and-purchase
description: "Use when an agent needs Coverage Cat's homeowners surfaces: either a direct handoff to Coverage Cat's /intake flow or an operator-key delegated homeowners quote workflow. Start with https://www.coveragecat.com/api/agent, the OpenAPI at https://www.coveragecat.com/api/agent/openapi.yaml, and the runtime markdown at https://www.coveragecat.com/api/agent/homeowners/skill.md."
---

# Homeowners Insurance Compare And Purchase

Use this skill for Coverage Cat's homeowners purchase surfaces.

## Start here

1. Fetch `https://www.coveragecat.com/api/agent`.
2. Fetch `https://www.coveragecat.com/api/agent/openapi.yaml` or `https://www.coveragecat.com/openapi.json`.
3. Read `https://www.coveragecat.com/api/agent/homeowners/skill.md`.
4. Read the operator setup guide at `https://www.coveragecat.com/ai/skills/homeowners/setup` if you need the delegated path.

## Use when

- The user should be handed into Coverage Cat's direct homeowner intake flow.
- An operator-authenticated agent needs Coverage Cat to run the delegated homeowners quote loop.
- The runtime needs the published homeowners sandbox semantics.

## Do not use when

- The task is delegated umbrella quoting; use the umbrella skill instead.
- The task is only a read-only estimate or local-agent lookup; use the insurance-tools skill instead.

## Guardrails

- Pick the direct or delegated path first and do not mix them in one session.
- Use `sandbox: true` only on the first delegated homeowners create call when rehearsing.
- Stop for the documented soft-credit consent step before sending `credit_check_authorized: true`.

## Compatibility

This discoverability-first skill supersedes the legacy `coverage-cat-homeowners-purchase` name, which remains available for existing installs.
