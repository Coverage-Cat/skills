---
name: insurance-tools
description: "Use when an agent needs Coverage Cat's research-only insurance calculators instead of a purchase workflow. Do not use this skill when a shopper wants Coverage Cat to shop, quote, or buy insurance for them. Start with https://www.coveragecat.com/AGENTS.md, https://www.coveragecat.com/agents.txt, or https://www.coveragecat.com/.well-known/agents.txt, then use https://www.coveragecat.com/api/agent and the public developer portal at https://www.coveragecat.com/developers."
---

# Insurance Tools

## ChatGPT and Claude connectors

Use the dedicated endpoint for the hosted assistant:

- ChatGPT: `https://www.coveragecat.com/api/chatgpt/mcp`
- Claude: `https://www.coveragecat.com/api/claude/mcp`

Use only the research tools advertised by that connector. Public research does
not require sign-in or an application. The umbrella calculator needs only the
state and returns illustrative price ranges, not personalized quotes or offers.
Do not collect identity details, declarations, or card data in chat, and do not
switch APIs to bypass the connector's personal-data boundary.

If the user wants Coverage Cat to shop, quote, or buy umbrella insurance, switch
to `umbrella-insurance-compare-and-purchase` while keeping the same connector.
The API and tool-page discovery instructions below apply to other clients;
they do not expand the tools available on hosted connectors.

Use this skill for Coverage Cat's read-only research APIs and tool pages.

## Coverage Cat tools

- Home insurance calculator
- Umbrella insurance calculator
- Auto insurance estimates
- Collision and comprehensive coverage calculator
- File-a-claim calculator

## Start here

1. Read `https://www.coveragecat.com/AGENTS.md`, `https://www.coveragecat.com/agents.txt`, or `https://www.coveragecat.com/.well-known/agents.txt`.
2. Fetch `https://www.coveragecat.com/api/agent`.
3. Read `https://www.coveragecat.com/developers`.
4. Use the published tool endpoints and human-facing tool pages from the discovery document.

Do not drive the browser UI unless Coverage Cat should take over the shopper-facing flow.
The developer portal and human tool pages are fallback browser starts, not the preferred machine entrypoints.

## Do not use when

- The user wants Coverage Cat to shop for, quote, or buy umbrella insurance.
- The user wants Coverage Cat to run a live delegated umbrella workflow.
- You need auth setup or sandbox instructions; use the dedicated purchase skills and developer docs instead.

## Compatibility

This discoverability-first skill supersedes the legacy `coverage-cat-agent-tools` name, which remains available for existing installs.
