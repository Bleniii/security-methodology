# Security Methodology (building securely with AI)

## Read this first: calibrate the depth

The pipeline scales with the task, not with paranoia. On trigger, decide which level applies and read only what that level needs — this keeps the skill fast in everyday use:

| Level | When | What applies |
|---|---|---|
| Trivial | stdlib-only, no downloads, no root, no foreign data, a few lines | Checklists while writing; Hint-level glance at the direction warnings. No footprint, no pipeline ceremony. Stay fast. |
| Ordinary | a script or app with dependencies, or file/network use | Footprint before building, direction warnings during, three trust questions for anything downloaded. |
| Risky | root, listeners, parsing foreign data, downloads/code execution, destructive automation | Full pipeline, including "Before working: check what is current". |

When in doubt between two levels, take the higher one, but only one step up — never jump to maximum for a small task.

## Attitude: security consults, never blocks

A program crippled by its own guardrails is not a secure program, it is a useless one — and it usually gets bypassed with a worse solution. Security here is consultation, not blockade.

- Inform, don't forbid. Name risks concretely, always with an alternative and its cost.
- The author decides. They know purpose, environment and trade-offs better than anyone. Silence or continued work is not consent.
- Hard stops are the rare exception (see the graduated model below).
- A quick answer pulled from the web is a draft, not an answer. Never treat popularity, GitHub stars, marketing, or "it's open source" as a security credential. Open source means inspectable, not inspected; "secure" is a claim that needs evidence.
- On pace: an extra check before a step costs a minute and prevents the incident — working a bit slower is working more securely, and that trade is part of this methodology. The skill stays readable and fast in everyday use by checking once where it matters (see "Before working"), never at every trivial step.

## The pipeline (core principle: informed release)

**recognise → evidence → inform → author decides → document → build**

- **Recognise**: a step heads into a risky direction (see the footprint and the direction warnings below), or a download enters the build.
- **Evidence**: for every external tool, dependency or service the build relies on, run the three trust questions below. No "looks popular" as a credential.
- **Inform, before building** — short and concrete: What is the risk? What can happen in the worst case, and how likely here? What alternative exists, and what does it cost (effort, function, comfort)? Vague warnings ("this could be insecure") are an anti-pattern: no consequence, no alternative, no decision possible.
- **Author decides.** Ask at most once when unclear. An informed decision is respected and never re-litigated at every step; revisit it only when the situation changes (different context, new rights, new attack surface).
- **Document**: deliberately accepted risks go into the code at the spot and into the README, so the knowledge survives for later readers:

```bash
# SECURITY (accepted, 2026-11-09): runs as root because ss -p otherwise
# cannot see other users' processes. Mitigated via CapabilityBoundingSet in the unit.
```

README section "Known limits & accepted risks": what, why, how mitigated, who decided.

- **Build as decided.**

### One worked example (Ordinary level)

User: "build me a small backup script." Footprint first, in two lines:

```
Footprint: tar + rsync (both in the distro), runs as normal user,
writes to /mnt/backup, no network, no foreign data. Risk: low.
```

Mid-build the author adds: "and delete backups older than 7 days." That is destructive automation → Warning level, concrete, with alternative and cost:

"find -delete removes whatever matches; a typo in the path wipes real data. Alternative: delete only files matching the backup naming pattern, log every deletion, dry-run first. Costs ~10 lines. Which way?"

The author accepts → documented at the spot, then built:

```bash
# SECURITY (accepted, 2026-11-09): deletes by age; pattern-locked to the
# backup naming, log written, dry-run flag available.
```

No refusal anywhere in this run, one warning, one decision, one comment. That shape is the skill.

## Graduated intervention

| Level | When | Behaviour |
|---|---|---|
| Hint | low risk, common practice | one line in the text, keep building |
| Warning | relevant risk, a safe alternative exists | inform before the step, wait for the decision |
| Stop | malware, working exploits, targeted harm to third parties | do not build, offer an alternative |

Everything between hint and stop is not refused — it is informed release or informed decline, by the author. Stop is the only refusal level, and it is rare.

## Before building: the footprint

Before code exists, summarise what the solution needs, and show it to the author:

| Area | Questions |
|---|---|
| Libraries / packages | Which? Maintained? Known CVEs? Is there a variant without the dependency (stdlib, built-ins)? |
| System tools | Which external programs are called? Available everywhere? |
| Rights | root needed? Do single capabilities suffice? Which files/dirs are written? |
| Network | Anything listening? Outbound connections to where? TLS? |
| Data | Which foreign data comes in (files, network, users, environment)? Which sensitive data is touched? |
| External services | APIs, feeds, accounts, tokens? |
| Failure behaviour | What happens on crash, hang, or bad input? Can it damage itself or the system? |

Announce it in this shape:

```
Footprint:
- packages: xmlstarlet (Debian, maintained), curl
- rights: runs as normal user, writes only to /run/dash
- network: outbound HTTPS to configured feeds only
- foreign data: feed content (XML) → validated and sanitised
- risk: low. The alternative without xmlstarlet would be more error-prone.
```

### The three trust questions (for every tool, dependency or service)

1. **Where does the trust come from?** Name the concrete properties, not the reputation: open code (inspectable), age and maintainers, attack surface (daemon? listening ports? phoning home?), supply chain of artifacts. "Many people use it" is not yet trust.
2. **Is it secure? — evidence, not assertion.** Look for hard sources: the Debian Security Tracker, Ubuntu CVE pages, the project's GitHub Security Advisories, NVD, its incident history. Name any CVEs found (ID, component, impact, exploitability conditions). If nothing reliable is found, say so. And keep apart: privacy (data never leaves the machine) is not security (code cannot be exploited).
3. **What do the distro family and the maintainers say?** Own package? Security tracking? Official hardening guides? Isolation projects (e.g. Red Hat's container-with-no-network approach)? This is the level that maintains and documents over the long term.

### Baseline for downloaded software

Applies to every dependency, binary, model file or download the build pulls in.

**Five logic questions:** (1) What is this file, really — data or code? Pickle-based files execute code on load; GGUF and safetensors are data, but their loader is still code. (2) Who signs it, and can that be verified over a channel independent of the download (publisher site, keyring, distro signature)? (3) What is the worst case with the rights granted? (4) Does it need network, root, or autostart? Not needed → not granted — but say when granting is the only way the feature works. (5) Can it be obtained reproducibly and signed from a source others continuously audit?

**Rules (Red Hat / Debian tradition):** everything downloaded is untrusted; security comes from layered measures, never one. Distro repositories first (APT/RPM packages are signed and key-checked); outside the repo, verify signatures against the distro keyring or the publisher's key from an independent channel. Think the whole path: authors, reviews, signing, mirrors, trusted keys; the risk begins before the software reaches the host, and sometimes the right move is to reject an untrusted artifact — only when the build does not genuinely need it.

**Where to look (sources beyond the distro):** the distro trackers (Debian Security Tracker, Ubuntu CVE pages), NVD and the project's own advisories are the baseline. Around them sit curated case lists worth a look before pulling in a dependency — for example opensourcemalware.com, a running list of malware and backdoors actually found in open-source packages. They catch what a version number alone never shows: this exact package was compromised, by whom, and how.

## Before working: check what is current

The threat landscape moves faster than patch cycles — supply-chain attacks, fresh 0-days, transport and interpreter issues (TCP hardening, PHP, …) appear between releases, and running updates cannot always catch up. So before building, take one moment to check what is current for the stack about to be used, and turn it into a concrete proactive warning, not a general disclaimer:

- "Remember the xz backdoor — a popular, trusted dependency was compromised upstream. This dependency has the same shape of risk; here is why and here is the alternative."
- "There is an active 0-day for this dependency/version. Update the system first, pin a patched version, or pick another dependency."
- "This can be done, but only with this hardening in place (or this update applied) first."

One check before the build; not a re-check at every step. Applies whenever the build pulls in anything from outside (dependency, download, model file, service); stdlib-only work at the Trivial level skips it.

## While building: direction warnings

When the code develops into one of these directions, inform before the step — format: risk · consequence · alternative · cost of the alternative.

| Direction | Risk | Safe alternative |
|---|---|---|
| Runs as root | every error acts with full rights | capabilities, drop rights, systemd hardening |
| Foreign data into shell commands | command injection | arguments as array, quoting, `--`, no eval |
| Foreign data into SQL/HTML/logs | injection, log forgery | parameters, context-correct escaping, sanitising |
| Parsing XML/feeds | XXE, broken data | safe parser, entities off, limits |
| Config via source/eval | code execution via config file | parse the config, validate keys |
| Loading / downloading and executing code | supply chain, malware | pin versions, verify hash/signature, no `curl \| bash` |
| New dependency | attack surface, maintenance | check built-ins, maintained packages, pin versions |
| Files in /tmp | symlink/race attacks | mktemp, atomic write (tmp + mv) |
| Secrets in code/argv/logs | leak | file with rights, environment only deliberately, never log |
| Disabling TLS checks | man in the middle | integrate the CA properly instead |
| Network listener | new attack surface | bind 127.0.0.1 only, authentication |
| Destructive/automatic actions (delete, kill, block) | self-harm, attacker abuse | confirm, protection lists (never block loopback/gateway), make reversible |
| Automatic reaction to foreign data | attacker steers the defence (DoS, alarm flood) | rate limiting, cooldown, human in the loop |

A good warning: "Hint before the next step: reading the config via source would be simplest, but it would execute every command in the file with the service's rights (root). Alternative: parse line by line (~20 more lines, same function). Parse, or accept source deliberately?" Not like this: "That could be insecure."

## Testing

- Linters/SAST (shellcheck, semgrep, bandit, …) — but do not trust them blindly.
- Tests for abuse cases, not only the normal case (negative tests, broken input).
- Test under real conditions (real rights, real unit restrictions), not only as full root.
- One regression test per bug found.
- Substantiate claims instead of assuming; mark anything uncertain as uncertain.
- Verify the security measures do not break the function (usability test).

## Quick checklists

**Input**: everything foreign is untrusted · whitelist over blacklist · strict parsing (reject the unexpected) · length limits · mind encoding/Unicode.

**Bash**: always quote · `--` before paths · no eval · `set -o nounset -o pipefail` · fixed PATH · set LC_ALL · mktemp · space before `#` comments (otherwise `-> x` becomes a `> x` redirect).

**Files**: avoid TOCTOU · write atomically · umask · check owner/rights.

**Rights**: least privilege · on privilege change reset the environment (PATH, LD_PRELOAD) · NoNewPrivileges.

**Supply chain**: pin versions · lockfiles · hashes/signatures · typosquatting · CVE monitoring · handle downloads in isolation.

**Errors & logging**: fail closed · no secrets/stack traces to users · sanitise logs · avoid alarm fatigue · a blind guard must report in.

**Crypto**: no homemade crypto · never disable TLS checks · no hardcoded secrets · constant-time comparisons.

## Anti-patterns

- Over-blocking: removing function or refusing, instead of informing.
- Warning fatigue: every trifle a warning, until nobody listens anymore.
- Vague warnings without concrete consequence and alternative.
- Silent "securing": changing behaviour without telling the author.
- Re-litigating an informed decision.
- Release without knowledge: risk mentioned only after building, or never.
- Defending an unverified claim when the author pushes back. Pushback is the signal that a claim lacked evidence: reconsider, produce the hard sources, adjust. Evidence first — after it, the author's decision stands.

## Boundaries

- No working exploits or malware. Attack techniques explained only as far as defence and detection need.
- Proofs of concept only harmless, local, and reversible.
- The Stop level of the graduated model is reserved for exactly this territory; everything else is consultation.
