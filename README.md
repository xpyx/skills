# skills

Agent skills that constrain how an agent works, not just what it knows.

Most skills add knowledge — an API reference, a framework's conventions. These
add rules: what the agent may do, what it must hand back to you, and where it
has to stop. They are behavioural constraints, and they are written to hold up
when the agent has a plausible reason to ignore them.

## Skills

- **[hands-on](plugins/hands-on/skills/hands-on/)** — you write the core business
  logic by hand; the agent scaffolds, writes tests, reviews, and never crosses
  that line.

## Install

```
/plugin marketplace add xpyx/skills
/plugin install hands-on@xpyx-skills
```

The first command registers this repo as a plugin marketplace; the second
installs a single skill from it. Run `/plugin marketplace update xpyx-skills`
to pick up later changes.

To install without Claude Code's plugin system, copy a skill directory from
`plugins/<name>/skills/` into your agent's skills directory, or point that
directory at it.

**Read [SECURITY.md](SECURITY.md) before installing.** Skill files are
executable instructions, not documentation, and that changes how you should
review them — especially if you track this repo directly and take updates
automatically.

## Layout

Each skill ships as its own plugin, so you install only what you want:

```
.claude-plugin/marketplace.json    # the catalogue
plugins/
  hands-on/
    .claude-plugin/plugin.json     # plugin manifest
    skills/hands-on/
      SKILL.md                     # the rules that must always hold
      references/                  # detail, loaded on demand
      tests/                       # scenarios showing the skill under pressure
```

Contributions welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).
