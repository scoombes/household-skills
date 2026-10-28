# HouseHold Skills

Custom agent skills for HouseHold development and operations, plus useful resources for writing and maintaining them.

This repository is the source of truth for three HouseHold skills and their ten supporting guides. Edit them here first, then install them with `hous skills sync` (housetools), which copies the selected skills into `~/.agents/skills`. Never edit the installed copies directly: `hous skills status` reports such edits as conflicts and `hous skills sync` refuses to overwrite them until the change is made here.

## Skills

| Skill | Purpose | Supporting guides |
| --- | --- | --- |
| [Operating HouseHold AWS](skills/operating-household-aws/SKILL.md) | HouseHold AWS operating rules and approval procedures. | [Approval mode](skills/operating-household-aws/reference/approval-mode.md), [cards and stops](skills/operating-household-aws/reference/cards-and-stops.md), [checks](skills/operating-household-aws/reference/checks.md), [DNS](skills/operating-household-aws/reference/dns.md), [failed stacks](skills/operating-household-aws/reference/failed-stack.md), and [traps](skills/operating-household-aws/reference/traps.md). |
| [Orca Pipeline](skills/orca-pipeline/SKILL.md) | HouseHold's per-story development and review workflow. | [Role responsibilities](skills/orca-pipeline/reference/roles.md), [panel review](skills/orca-pipeline/reference/panel-review.md), [panel coordinator rules](skills/orca-pipeline/reference/panel-coordinator.md), and [test standard](skills/orca-pipeline/reference/test-standard.md). |
| [Coordinating Epics](skills/coordinating-epics/SKILL.md) | HouseHold's coordination of work across multiple stories. | Shares the pipeline's [role responsibilities](skills/orca-pipeline/reference/roles.md). |

The `orca-cli`, `orchestration`, and `computer-use` skills belong to Orca IDE and remain external dependencies. Other references to `household-infra` and `housetools` remain in those repositories.

AWS's upstream guide library stays outside this repository. The AWS skill explains how to reuse the existing local copies and [where to download needed documents if missing](skills/operating-household-aws/SKILL.md#external-aws-guides). No third-party AWS guides, evaluation files, or tests are included.

## Repository structure

```text
household-skills/
├── README.md                 Purpose, authoring guidance, and resources
├── AGENTS.md                 Instructions for repository contributors
└── skills/                   One directory per selected skill
```

Each skill follows the [Agent Skills specification](https://agentskills.io/specification). Its directory name matches the `name` in its `SKILL.md` frontmatter:

```text
skills/<skill-name>/
├── SKILL.md                  Required: name, description, and instructions
├── references/               Optional: supporting documentation
├── scripts/                  Optional: executable helpers
├── assets/                   Optional: templates and output resources
└── agents/openai.yaml        Optional: Codex metadata and dependencies
```

Create optional files only when the workflow needs them. Existing skills may use different reference-directory names; preserve working links when importing them.

## Writing and maintaining skills

Start with a concrete, repeatable HouseHold task. Describe what the skill does and when it applies in the frontmatter; put the workflow, required inputs, and expected outcome in the body. Keep instructions focused on knowledge that improves the agent's decisions.

Keep the main instructions concise. Link supporting material from `SKILL.md` and explain when to read it. Document any required tools or host-specific behavior, especially where Codex, Claude Code, or Orca differ. See [OpenAI's authoring guidance](https://developers.openai.com/plugins/build/skills) and [Anthropic's authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

Before importing an existing skill, review its complete folder, dependencies, scripts, and references. Include the selected HouseHold documents and preserve working relative paths. Keep upstream AWS guides outside this repository and preserve required notices with downloaded copies. Update the skill catalogue above when adding a selected skill.

Validate format and file references, then try representative requests: one that should invoke the skill, a nearby request that should not, and incomplete input. Check the actual result and any script behavior. [OpenAI's evaluation guide](https://developers.openai.com/blog/eval-skills) describes a way to make these checks repeatable.

## Useful resources

| Resource | What it covers |
| --- | --- |
| [Anthropic: skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) | Concise instructions, progressive disclosure, examples, and evaluation. |
| [Anthropic: extend Claude Code with skills](https://code.claude.com/docs/en/skills) | Discovery, installation locations, invocation, frontmatter, and supporting files. |
| [OpenAI: build skills for ChatGPT and Codex](https://learn.chatgpt.com/docs/build-skills) | Skill creation, discovery, local skill locations, and Codex metadata. |
| [OpenAI: skill authoring for plugins](https://developers.openai.com/plugins/build/skills) | Workflow boundaries, references, scripts, tool dependencies, and testing. |
| [OpenAI: testing skills with evaluations](https://developers.openai.com/blog/eval-skills) | Testing activation and observable outcomes with representative prompts. |
| [Agent Skills specification](https://agentskills.io/specification) | The shared format, naming rules, frontmatter, and directory conventions. |

Resource links checked on 12 October 2026. Storing a skill under `skills/` does not by itself make it available to agents: `hous skills sync` installs the selected skills into `~/.agents/skills` (Codex reads that directory; Claude Code reads it through the `~/.claude/skills/<name>` links, which `hous skills status` checks but never creates). `hous skills status` shows what differs and `hous skills sync --dry-run` previews a sync without writing. The sync is local and one-way: it never fetches, pulls or pushes, installs the working tree of this checkout, refuses a checkout that is behind its last-fetched `origin/main`, and leaves the external AWS guides (`vendor/`), evaluation files and Orca-owned skills in place.
