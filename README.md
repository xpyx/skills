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

**Read [SECURITY.md](SECURITY.md) before installing.** Skill files are
executable instructions, not documentation, and that changes how you should
review them — especially if you track this repo directly and take updates
automatically.

### Claude Code

```
/plugin marketplace add xpyx/skills
/plugin install hands-on@xpyx-skills
```

The first command registers this repo as a plugin marketplace; the second
installs a single skill from it. Run `/plugin marketplace update xpyx-skills`
to pick up later changes.

### GitHub Copilot

Copilot CLI reads the same manifests — `.claude-plugin/` is one of the
locations it looks in for `marketplace.json` and `plugin.json` — so this repo
works as a Copilot marketplace with no extra config:

```
copilot plugin marketplace add xpyx/skills
copilot plugin install hands-on@xpyx-skills
```

Run `copilot plugin update hands-on` to pick up later changes. Installing
straight from the repo (`copilot plugin install xpyx/skills:plugins/hands-on`)
also works, but Copilot warns that direct installs are deprecated in favour of
`plugin@marketplace` — prefer the two commands above.

### Kiro

Kiro imports a skill from a public GitHub URL. In the Agent Steering & Skills
panel, choose **+** → *Import a skill* → *GitHub*, and paste the path to the
skill directory:

```
https://github.com/xpyx/skills/tree/main/plugins/hands-on/skills/hands-on
```

The URL has to point at a subdirectory, not the repo root. Kiro copies the
skill into `.kiro/skills/` (workspace) or `~/.kiro/skills/` (global), so
re-import to pick up later changes. Custom agents don't load skills
automatically — reference the skill from the agent's `resources` field with a
`skill://` URI.

There is no Kiro *power* for this repo: powers expect a `plugin.json` at the
repository root, which would mean one repo per skill.

### Any other agent

Copy a skill directory from `plugins/<name>/skills/` into your agent's skills
directory, or point that directory at it. Each skill is a self-contained
`SKILL.md` plus its `references/` and `tests/`, with no dependency on the
plugin manifests around it.

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

Claude Code and Copilot CLI both read these two manifests, which is why one
layout serves both.

Contributions welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).
