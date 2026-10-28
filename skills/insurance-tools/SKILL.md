---
name: insurance-tools
description: "Use when an agent needs Coverage Cat's research-only insurance calculators or homeowners-agent finder instead of a purchase workflow. Do not use this skill when a shopper wants Coverage Cat to shop, quote, or buy insurance for them. Start with https://www.coveragecat.com/AGENTS.md, https://www.coveragecat.com/agents.txt, or https://www.coveragecat.com/.well-known/agents.txt, then use https://www.coveragecat.com/api/agent and the public developer portal at https://www.coveragecat.com/developers."
---

# Insurance Tools

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

Use this skill for Coverage Cat's read-only research APIs and tool pages.

## Coverage Cat tools

- Home insurance calculator
- Umbrella insurance calculator
- Auto insurance estimates
- Collision and comprehensive coverage calculator
- File-a-claim calculator
- Homeowners agent finder

## Start here

1. Read `https://www.coveragecat.com/AGENTS.md`, `https://www.coveragecat.com/agents.txt`, or `https://www.coveragecat.com/.well-known/agents.txt`.
2. Fetch `https://www.coveragecat.com/api/agent`.
3. Read `https://www.coveragecat.com/developers`.
4. Use the published tool endpoints and human-facing tool pages from the discovery document.

Do not drive the browser UI unless Coverage Cat should take over the shopper-facing flow.
The developer portal and human tool pages are fallback browser starts, not the preferred machine entrypoints.

## Do not use when

- The user wants Coverage Cat to shop for, quote, or buy umbrella or homeowners insurance.
- The user wants Coverage Cat to run a live delegated umbrella or homeowners workflow.
- You need auth setup or sandbox instructions; use the dedicated purchase skills and developer docs instead.

## Compatibility

This discoverability-first skill supersedes the legacy `coverage-cat-agent-tools` name, which remains available for existing installs.
