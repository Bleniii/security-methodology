# Changelog

All notable changes to this skill are documented here.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.0.0] - 2026-11-10

### Added
- Calibrated depth on trigger: Trivial / Ordinary / Risky levels, so the skill stays fast on small tasks and thorough on risky ones.
- The pipeline: recognise → evidence → inform → author decides → document → build (informed release).
- Graduated intervention: Hint / Warning / Stop, with Stop as the only refusal level.
- Pre-build footprint (packages, rights, network, foreign data, failure behaviour) with template.
- Three trust questions for every tool, dependency or service.
- Baseline for downloaded software: five logic questions plus Red Hat / Debian rules.
- Sources beyond the distro: Debian Security Tracker, Ubuntu CVE pages, NVD, curated case lists such as opensourcemalware.com.
- Current-threat check before building (0-days, supply-chain attacks, transport/interpreter issues), with concrete warning examples (xz backdoor).
- Direction warnings table: 14 risky directions with safe alternatives and their cost.
- Worked example (Ordinary level): a backup script run end to end.
- Testing guidance, quick checklists (input, bash, files, rights, supply chain, errors & logging, crypto), anti-patterns, boundaries.

### Notes
- Origin: a real incident where a fast web answer missed open CVEs in a local-AI stack; pushback produced the evidence, the evidence produced the methodology.
- Design principle: security consults, never blocks. Slower is safer, but the check happens once where it matters, not at every step.
