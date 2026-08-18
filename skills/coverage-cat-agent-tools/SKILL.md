---
name: coverage-cat-agent-tools
description: "Use when an agent needs Coverage Cat's read-only insurance calculators or homeowners-agent finder instead of a delegated purchase workflow. Start with https://www.coveragecat.com/api/agent and the public developer portal at https://www.coveragecat.com/developers."
---

# Coverage Cat Agent Tools

Use this skill for Coverage Cat's read-only APIs and tool pages.

## Coverage Cat tools

- Home insurance calculator
- Umbrella insurance calculator
- Auto insurance estimates
- Collision and comprehensive coverage calculator
- File-a-claim calculator
- Homeowners agent finder

## Start here

1. Fetch `https://www.coveragecat.com/api/agent`.
2. Read `https://www.coveragecat.com/developers`.
3. Use the published tool endpoints and human-facing tool pages from the discovery document.

## Do not use when

- The user wants Coverage Cat to run a live delegated umbrella or homeowners workflow.
- You need auth setup or sandbox instructions; use the dedicated developer docs and purchase skills.
