# Contributing

Thanks for your interest in contributing to Spiclaw Skills!

## Adding a new skill

### 1. Create the skill folder

```
skills/<your-skill-name>/
├── SKILL.md      # required — metadata + full prompt
├── README.md     # optional — human-readable docs
└── ...           # any additional files your skill needs
```

### 2. Write the `SKILL.md`

This is the core of your skill. It starts with a YAML frontmatter block:

```yaml
---
name: your-skill-name
description: Short description of what the skill does and when it should trigger.
---
```

Then document the full workflow the agent should follow: inputs, steps, tools used, output format, and error handling.

### 3. Keep it focused

- One skill = one job. Don't bundle unrelated features.
- Write clear trigger descriptions so the agent knows when to use it.
- Handle errors explicitly — don't let the agent guess.

## Improving an existing skill

- Bug fixes and clarity improvements are always welcome.
- If changing behavior, explain **why** in the PR description.

## Submitting

1. Fork the repo
2. Create a branch: `feat(<skill-name>): short description`
3. Commit following [conventional commits](https://www.conventionalcommits.org/)
4. Open a PR against `main`

## Code style

- English for all code, comments, and docs
- Keep prompts concise — agents pay per token
- No hardcoded secrets or API keys

## Questions?

Open an issue — we'll figure it out together.
