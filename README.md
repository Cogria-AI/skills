# Cogria Skills

A collection of [Agent Skills](https://code.claude.com/docs/en/skills) for AI coding assistants. Each skill lives in its own folder with a `SKILL.md` entry point and optional `references/`.

## Skills

| Skill | What it does |
|-------|--------------|
| [agent-dev-guide](agent-dev-guide/) | Right-size AI agent architecture: decide which capabilities (tools, loop, MCP, RAG, memory, multi-agent, guardrails, evals…) a project actually needs, and leave out the rest. |

## Install

Clone the repo, then link the skills you want into your skills directory.

```bash
git clone https://github.com/Cogria-AI/skills.git ~/cogria-skills

# Personal (all projects)
ln -s ~/cogria-skills/agent-dev-guide ~/.claude/skills/agent-dev-guide

# Or per project
mkdir -p .claude/skills
ln -s ~/cogria-skills/agent-dev-guide .claude/skills/agent-dev-guide
```

Copy the folder instead of symlinking if you want a frozen version. Run `git pull` in the clone to update linked skills.

## Layout

```
<skill-name>/
├── SKILL.md          # frontmatter (name, description) + core instructions
└── references/       # detail loaded on demand
```

## License

MIT
