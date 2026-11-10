# security-methodology

**How to build securely with AI: consults instead of blocks, informed decisions, proportionate guardrails.**

A portable skill file (SKILL.md) for AI assistants. It encodes a working methodology for building, scripting, installing and deploying things *with* an AI, without getting crippled by paranoia and without trusting blindly.

## Why

The skill was born from a real incident: someone asked their AI assistant for a lightweight, secure local AI setup and got a fast, confident answer pulled from the web. Only pushback uncovered open CVEs, a vulnerable loader and missing hardening. The lesson became a methodology, and the methodology became this skill:

> A quick answer pulled from the web is a draft, not an answer. "Open source" means inspectable, not inspected. Security that kills the function is a failure.

## What it does

- **Calibrated depth** — on trigger, the assistant decides: trivial task (stay fast), ordinary task (footprint + warnings), risky task (full pipeline). Scales with the task, not with paranoia.
- **The pipeline** — recognise → evidence → inform → author decides → document → build. Informed release instead of refusal.
- **Graduated intervention** — Hint / Warning / Stop. Stop is the only refusal level, and it is rare.
- **The footprint** — a short summary of packages, rights, network, foreign data and failure behaviour *before* code exists.
- **Three trust questions** — where does the trust come from, what is the evidence (CVEs, trackers, incident history), what do the distro maintainers say. For every dependency, tool or service.
- **Baseline for downloaded software** — five logic questions plus the Red Hat / Debian tradition: everything downloaded is untrusted, security is layered, distro repos first.
- **Current-threat check** — supply-chain attacks, fresh 0-days, transport and interpreter issues move faster than patch cycles; one check before building, turned into concrete warnings (xz-backdoor lessons, opensourcemalware.com).
- **Direction warnings** — 14 risky directions (root, injection, /tmp, secrets, listeners, destructive automation, …) with the safe alternative and its cost.
- **Checklists, anti-patterns, boundaries** — including the meta-rule: *pushback is the signal that a claim lacked evidence; produce the sources, then the author's decision stands.*

## Using it

The file is plain markdown and works with any assistant that supports instruction files or skills:

- **Claude / Claude Code**: drop it in as a skill or paste it into `CLAUDE.md` / project instructions.
- **ChatGPT / others**: paste the body into custom instructions or a project instruction file, or reference it at the start of a session.

Suggested description (the trigger line, ~970 characters):

```
How to build securely with AI: consults instead of blocks, informed decisions, proportionate guardrails. Use this skill whenever the user wants to build, script, install or deploy anything (scripts, apps, tools, automations, cron/systemd jobs), adds a dependency or downloads software or model files, asks whether a tool, package, dependency or source is safe or trustworthy, or works with root rights, network listeners, parsing of foreign data or destructive automation — even when they never mention security. Runs the pipeline recognise → evidence → inform → author decides → document → build with graduated intervention (hint / warning / stop), a pre-build footprint, three trust questions, a baseline for downloaded software, current-threat checks (0-days, supply chain, xz-backdoor lessons, opensourcemalware.com), direction warnings, quick checklists and anti-patterns. Usability stays intact: security that kills the function is a failure, never refuse to build.
```

## Status

v1.0.0 — in active use. See [CHANGELOG.md](CHANGELOG.md).

## License

MIT — see [LICENSE](LICENSE).
