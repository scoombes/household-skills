# Contributing to HouseHold Skills

This repository holds selected custom HouseHold skills. Keep the authoring resource links in `README.md`.

- Put each selected skill in `skills/<skill-name>/SKILL.md`, with matching `name` and a clear `description` in YAML frontmatter.
- Edit skills in this repository first, then install them with `hous skills sync` (preview with `hous skills sync --dry-run`, inspect with `hous skills status`). Never edit `~/.agents/skills` directly: a direct edit becomes a conflict that blocks the next sync until it is made here. This includes AWS lessons: a new rule in `skills/operating-household-aws/reference/traps.md` is written here and synced, not patched into the installed copy.
- Import only skills Sean has selected. Include their HouseHold supporting documents and preserve dependencies.
- Keep third-party AWS guides outside this repository. Reuse existing local copies; the AWS skill records where to download a document if it is missing and needed. Do not import the AWS library or evaluation files.
- Keep instructions focused on the skill's actual workflow. Add references, helpers, or assets only when needed.
- Use relative paths for bundled resources and check that they resolve. Keep runtime-generated Orca guidance tied to the installed Orca version.
- This is a documentation-only repository. Do not add tests, a test runner, or test infrastructure. Review formatting, links, and resource paths when changing documents.
- Use `hous` for supported HouseHold tooling.
- Working files and scratch belong in `~/household/agent-work/<TICKET>/`; plans and reports belong in `~/household/epics/<TICKET>/`.
