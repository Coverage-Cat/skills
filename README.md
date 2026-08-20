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

- Developer portal: `https://www.coveragecat.com/developers`
- Discovery JSON: `https://www.coveragecat.com/api/agent`
- OpenAPI: `https://www.coveragecat.com/api/agent/openapi.yaml`

## Source of truth

- `docs/agents/`: canonical agent-facing OpenAPI and runtime markdown sources that Coverage Cat syncs into the private app repo at build time.
- `skills/`: installable public skill packages for agent runtimes and the Skills CLI.
