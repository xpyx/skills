# Security Policy

## Reporting a vulnerability

Please report security issues privately via
[GitHub Security Advisories](https://github.com/xpyx/skills/security/advisories/new)
rather than opening a public issue.

## Threat model

This repository contains **agent skills** — Markdown files that are loaded into
the context of an AI coding agent and interpreted as instructions. That makes
the threat model different from a typical library, and worth stating plainly.

**The content of this repo is executable.** A skill file is not inert
documentation. When an agent loads `SKILL.md`, the text becomes instructions the
agent will act on, with whatever tool permissions that agent has been granted —
file writes, shell commands, network access. There is no sandbox between a skill
file and the machine running the agent.

Consequences worth internalising:

- **A malicious pull request is a code-execution attempt**, even though it only
  touches Markdown. Reviewing a diff here carries the same weight as reviewing a
  patch to a build script.
- **Checking out a contributor's branch can be enough.** If your local clone is
  symlinked into your agent's skill directory (a common setup), switching to an
  untrusted branch makes that branch's instructions live in your next session. No
  merge required.
- **Prompt injection is the primary attack.** Instructions can be hidden in
  places human reviewers skim past: HTML comments, code fences, example
  transcripts, or text framed as something the agent should "note" or "remember".

## If you maintain a fork or install these skills

1. **Don't point your agent's skill directory at a working tree you use for
   development.** Keep a separate checkout that only ever advances to a reviewed
   commit, or copy files in deliberately after review.
2. **Review contributions on the web, not by checking them out**, unless you have
   isolated your skill path from your working tree.
3. **Audit your agent's standing permissions.** Broad allowlist entries such as
   `Bash(python3:*)` or `Bash(bun:*)` are unrestricted code execution, and an
   auto-accept edit mode removes the prompt that would otherwise surface an
   unexpected action.

## Scope

This repo ships no executable code, no dependencies, and no CI workflows.
GitHub Actions is disabled deliberately. If you see a pull request that adds a
workflow file, treat it as suspicious and report it.
