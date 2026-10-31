# Contributing to HouseHold Skills

This repository holds selected custom HouseHold skills. Keep the authoring resource links in `README.md`.

- Put each selected skill in `skills/<skill-name>/SKILL.md`, with matching `name` and a clear `description` in YAML frontmatter.
- Edit skills in this repository first, then install them with `hous skills sync` (preview with `hous skills sync --dry-run`, inspect with `hous skills status`). Never edit `~/.agents/skills` or `~/.claude/skills` directly: a direct edit becomes a conflict that blocks the next sync until it is made here. This includes AWS lessons: a new rule in `skills/operating-household-aws/reference/traps.md` is written here and synced, not patched into the installed copy.
- `hous skills` reads `~/household/household-skills` by default, not the checkout you are in. From an Orca worktree, preview with `hous skills sync --dry-run --source "$PWD"`. The normal install is after merge: update `~/household/household-skills` to `origin/main`, then run `hous skills sync`. Every skill keeps a closing "Changing this skill" section that points installed readers back here.
- Import only skills Sean has selected. Include their HouseHold supporting documents and preserve dependencies.
- Keep third-party AWS guides outside this repository. Reuse existing local copies; the AWS skill records where to download a document if it is missing and needed. Do not import the AWS library or evaluation files. The installed `~/.agents/skills/operating-household-aws/evals/evals.json` is local to the installation: it stays where it is, outside this repository and outside `hous skills sync`, and is not imported, edited here or overwritten. The AWS skill's own override already says never to add `Metadata.AWSToolsMetrics` or `com.aws.cloudformation.Context` to HouseHold templates (`skills/operating-household-aws/SKILL.md`, "Overrides of upstream AWS guidance"); do not add a second override or edit the vendor guides.
- Keep instructions focused on the skill's actual workflow. Add references, helpers, or assets only when needed.
- Use relative paths for bundled resources and check that they resolve. Keep runtime-generated Orca guidance tied to the installed Orca version.
- This is a documentation-only repository. Do not add tests, a test runner, or test infrastructure. Review formatting, links, and resource paths when changing documents.
- Use `hous` for supported HouseHold tooling.
- Working files and scratch belong in `~/household/agent-work/<TICKET>/`; plans and reports belong in `~/household/epics/<TICKET>/`.
