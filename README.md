# general-agent-skills
General-purpose agent skills for writing, communication, and content creation.

## Skills

| Skill | Description |
|-------|-------------|
| [humanizer](skills/humanizer/) | Removes AI-writing patterns and rewrites text to sound natural and human |

## Install

Clone this repo:

```bash
git clone https://github.com/CarbonMonkeyDev/general-agent-skills ~/.agents/skills/general-agent-skills
```

### OpenClaw

Add the skills path to your agent config in `~/.openclaw/openclaw.json`:

```json
{
  "skills": {
    "load": {
      "extraDirs": ["~/.agents/skills/general-agent-skills/skills"]
    }
  }
}
```

Restart the gateway or let it pick up changes automatically.

### Claude Code

Link skills into `~/.claude/skills/`:

```bash
mkdir -p ~/.claude/skills/
ln -sf ~/.agents/skills/general-agent-skills/skills/* ~/.claude/skills/
```

### Codex / OpenCode

You're all set after cloning to the shared skills root.

### Other agents

Follow your agent's docs for skills location.

## Update

Pull updates in the repo:

```bash
git -C ~/.agents/skills/general-agent-skills pull
```

OpenClaw: restart the gateway or let it pick up changes automatically.

Claude Code: re-link after pulling:

```bash
ln -sf ~/.agents/skills/general-agent-skills/skills/* ~/.claude/skills/
```

Other agents: follow their docs for update paths.

## Contributing

1. Create a new folder under `skills/<skill-name>/`
2. Add a `SKILL.md` with YAML frontmatter and instructions
3. Update this README's skill table
4. Open a pull request
