# CONTRIBUTING

Thanks for your interest in contributing to Spiclaw Skills!

## Adding a new skill

### 1. Create the skill folder

```
<your-skill-name>/
├── SKILL.md      # required — metadata + full prompt
├── README.md     # required — human-readable docs
├── CHANGELOG.md  # required — version history
└── ...           # any additional files your skill needs
```

### 2. Choose a slug

The folder name **is** the slug used on [ClawHub](https://clawhub.ai/). Slugs are **globally unique** — if someone else already published a skill with the same slug, yours will be rejected.

Rules:
- Lowercase, hyphens only (no underscores, no spaces)
- Max 64 characters
- Descriptive and concise (e.g., `workout-track`, `remind-myself`)
- The `name` field in `SKILL.md` frontmatter **must match** the folder name

The CI will check slug availability on your PR and **block the merge** if the slug is already taken. If that happens, rename your folder and update the `name` field accordingly.

### 3. Write the `SKILL.md`

This is the core of your skill. It starts with a YAML frontmatter block:

```yaml
---
name: your-skill-name
description: Short description of what the skill does. Use when <trigger context>.
---
```

Then document the full workflow the agent should follow: inputs, steps, tools used, output format, and error handling.

### 4. Write the `CHANGELOG.md`

Start with:

```markdown
# Changelog

## 1.0.0

Initial release.
```

Bump the version for each update. The top `## x.y.z` entry is used as the published version.

### 5. Keep it focused

- One skill = one job. Don't bundle unrelated features.
- Write clear trigger descriptions so the agent knows when to use it.
- Handle errors explicitly — don't let the agent guess.

## Improving an existing skill

- Bug fixes and clarity improvements are always welcome.
- If changing behavior, explain **why** in the PR description.
- Bump the version in `CHANGELOG.md`.

## CI checks

When you open a PR, the CI will automatically:

1. **Validate structure** — checks that `SKILL.md`, `README.md`, and `CHANGELOG.md` exist with required fields
2. **Check slug availability** — verifies your slug isn't already taken on ClawHub

Both checks must pass before the PR can be merged. After merge, the skill is automatically published to ClawHub.

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
