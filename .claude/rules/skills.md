# Skills conventions

## Quality checklist

Before committing a new or modified skill, validate it against `skill-build-helper/references/checklist.md`. Key points:

- Frontmatter: `name` matches folder, `description` includes "Use when...", `metadata.openclaw.requires` declares dependencies
- Body: workflow with numbered H3 steps, error handling section, 3+ examples, confirmation before state changes
- Files: `SKILL.md` + `README.md` + `CHANGELOG.md` all present

## README skills table

When adding or modifying a skill:

- **New skill**: add an entry to the skills table in `README.md` with status `![stable](https://img.shields.io/badge/stable-green)` or `![wip](https://img.shields.io/badge/WIP-orange)`
- **Status change**: update the badge (e.g. WIP → stable)
- **Removal**: remove the row from the table

The CI will block the PR if a skill directory exists but is not listed in the README table.
