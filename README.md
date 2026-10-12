# HouseHold Skills

Custom agent skills for HouseHold development and operations, plus useful resources for writing and maintaining them.

This repository will start with selected skills from the existing HouseHold setup. Selection is pending; no existing skills have been copied or installed yet.

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

Before importing an existing skill, review its complete folder, dependencies, scripts, and references. Preserve required notices and working relative paths. Add the skill to a catalogue here once it has been selected and imported.

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

Resource links checked on 12 October 2026. Follow the current host documentation when installing skills; storing them under this repository's `skills/` directory does not automatically make them available in other HouseHold repositories.
