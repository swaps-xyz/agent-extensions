# swaps-xyz/agent-extensions

Team-specific Claude Code skills for the Swaps team.

## Skills

| Skill | Description |
|---|---|
| [`swa-triage-sweep`](./skills/swa-triage-sweep/SKILL.md) | Sweep the SWA Linear triage backlog, propose priorities and project assignments, publish a Slack Canvas, and post to #swaps-leads |

## Adding these skills to your Claude Code setup

Skills live in `skills/<skill-name>/SKILL.md`. You can load them in two ways:

### Option 1 — Clone and symlink (recommended)

Clone this repo once, then symlink the skills directory into your global Claude skills location:

```bash
git clone git@github.com:swaps-xyz/agent-extensions.git ~/code/swaps-xyz/agent-extensions

# Symlink individual skills into your global skills dir
mkdir -p ~/.claude/skills
ln -s ~/code/swaps-xyz/agent-extensions/skills/swa-triage-sweep ~/.claude/skills/swa-triage-sweep
```

Pull updates any time with `git pull` — symlinks pick up changes immediately.

### Option 2 — Copy a skill directly

Copy the skill directory into your project's `.claude/skills/` folder or your global `~/.claude/skills/`:

```bash
cp -r skills/swa-triage-sweep ~/.claude/skills/swa-triage-sweep
```

### Verifying installation

After installing, Claude Code will surface the skill automatically. You can confirm it loaded by running:

```
/skills
```

and looking for the skill name in the list.

### Required MCPs

Some skills require MCP servers to be configured. Check the **Prerequisites** section of each `SKILL.md` for what's needed. For `swa-triage-sweep` you need:

- **Linear MCP** — for fetching triage issues and projects
- **Slack MCP** — for Canvas creation and message posting

## Adding a new skill

1. Create `skills/<skill-name>/SKILL.md` with YAML frontmatter:

```markdown
---
name: skill-name
description: One-line description — used by Claude to decide when to invoke this skill.
user_invocable: true
version: 1.0.0
---

# Skill Title

Skill instructions here.
```

2. Add a row to the Skills table in this README.
3. Open a PR.
