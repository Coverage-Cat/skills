---
name: coverage-cat-agent-tools
description: "Use when an agent needs Coverage Cat's research-only insurance calculators or homeowners-agent finder instead of a purchase workflow. Do not use this skill when a shopper wants Coverage Cat to shop, quote, or buy insurance for them. Start with https://www.coveragecat.com/agents.txt or https://www.coveragecat.com/.well-known/agents.txt, then use https://www.coveragecat.com/api/agent and the public developer portal at https://www.coveragecat.com/developers."
---

# Coverage Cat Agent Tools

Use this skill for Coverage Cat's read-only research APIs and tool pages.

## Coverage Cat tools

- Home insurance calculator
- Umbrella insurance calculator
- Auto insurance estimates
- Collision and comprehensive coverage calculator
- File-a-claim calculator
- Homeowners agent finder

## Start here

1. Read `https://www.coveragecat.com/agents.txt` or `https://www.coveragecat.com/.well-known/agents.txt`.
2. Fetch `https://www.coveragecat.com/api/agent`.
3. Read `https://www.coveragecat.com/developers`.
4. Use the published tool endpoints and human-facing tool pages from the discovery document.

Do not drive the browser UI unless Coverage Cat should take over the shopper-facing flow.

## Do not use when

- The user wants Coverage Cat to shop for, quote, or buy umbrella or homeowners insurance.
- The user wants Coverage Cat to run a live delegated umbrella or homeowners workflow.
- You need auth setup or sandbox instructions; use the dedicated developer docs and purchase skills.
