# Contributing

Contributions are welcome. Please read [SECURITY.md](SECURITY.md) first — skill
files are interpreted as instructions by an AI agent, so changes here behave more
like changes to executable code than to documentation.

## Ground rules

- **One skill per pull request.** Mixed PRs are harder to review carefully, and
  careful review is the only safety mechanism this repo has.
- **No GitHub Actions workflows.** Actions is disabled deliberately. A PR adding
  `.github/workflows/` will be closed.
- **No new dependencies, build steps, or scripts.** This repo is Markdown only.
- **Plain, visible prose.** No hidden HTML comments, zero-width characters, or
  instructions embedded in places a reviewer would skim past. Anything an agent
  is meant to act on must be readable in the rendered diff.

## Writing a skill

A skill lives in its own directory with a `SKILL.md` at the root:

```
my-skill/
  SKILL.md          # frontmatter (name, description) + the skill body
  references/       # detail files the skill points to, loaded on demand
  tests/            # scenarios demonstrating the skill under pressure
```

Keep `SKILL.md` short and make it carry the rules that must always hold. Push
detail into `references/` and cite it rather than restating it — duplicated rules
drift apart, and the copy that drifts is the one that gets followed.

Where a skill constrains agent behaviour, include test scenarios showing it
holding under pressure. A rule that has never been tested against a plausible
rationalisation is a rule that will not hold.

## Review expectations

Every PR is reviewed as a potential injection attempt. Expect questions about
wording that would seem innocuous in an ordinary docs PR — instructions that
broaden an agent's autonomy, that direct it to read or write outside the
repository, or that tell it to disregard other guidance. This is not a judgement
about you; it is the only review posture that makes sense given what these files
do.

## Licensing

Contributions are accepted under the [MIT License](LICENSE).
