# Contributing to Pathfinder

## Adding a New Skill

1. Create a directory under `skills/` with a `SKILL.md` file
2. Follow the existing pattern: frontmatter → Methodology → Steps → Verification Gates
3. Include at least 2 verification gates

## Adding a New Agent

1. Create a `.md` file in `agents/`
2. Include frontmatter: name, description, tools, model
3. Define: workflow, output format, critical rules

## Adding a New Command

1. Create a `.md` file in `commands/`
2. Include frontmatter with description
3. Define: what it does, example output

## Updating Documentation

Pathfinder uses the [company-docs](https://github.com/akshiitGpt/company-docs) repo as its knowledge base. To update:

1. Clone the company-docs repo separately
2. Add or edit markdown files in `knowledge-base/` following the existing front-matter format
3. Commit and push to company-docs — Pathfinder auto-pulls on each session

## Adding a New Repo to the Portfolio

1. Add a service deep dive in `company-docs/knowledge-base/services/{name}/`
2. Add a code-level guide in `company-docs/knowledge-base/repos/{name}.md`
3. Update `company-docs/knowledge-base/architecture/service-map.md`
4. Update the Connected Repositories table in `README.md`
5. Update domain area mapping in `skills/ticket-analysis/SKILL.md`
