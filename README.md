# Job Portfolio — Local Package (prepared 2026-09-08)

**Status: prepared locally for Hunter's own review. Not published, not
production-ready, not to be shared externally without a separate, explicit
go-ahead.**

## What this is

Four verified case studies drawn from real Claude Code Work Inbox missions on
this machine, prepared for Hunter to review and explain in his own words to an
AI-related employer. Nothing here is invented: every claim traces to a specific
commit, command, test result, or internal mission record, listed as evidence at
the end of each case study.

## How to read this

Start with whichever case study matches the conversation you're having. None of
the four is "the lead" — they're evenly weighted and can be read in any order.

- **[01 — The verification system](01-verification-system/case-study.md)**: the
  actual governance/verification machinery — how AI-executed work is kept
  honest before it's ever called "done."
- **[02 — AI-worker fleet supervision](02-fleet-supervision/case-study.md)**:
  that system in action, catching two real problems from two different coding
  agents.
- **[03 — Python-Inspector](03-python-inspector/case-study.md)**: a real piece
  of software — built, security-hardened, and validated against real projects.
- **[04 — Evidence-durability repair](04-evidence-durability/case-study.md)**: a
  real bug found in the verification system's own infrastructure, fixed, and
  hash-verified.

[interview-notes.md](interview-notes.md) has a short, plain-language version of
each, meant to practice from out loud.

## How Hunter's role is described throughout

As **AI orchestration and verification**: directing what gets built or checked,
making the judgment calls and the one real permission decision on record,
reviewing AI-executed output before accepting it, and deciding what counts as
actually fixed versus merely claimed. The code and configuration changes
themselves were drafted and executed by Claude Code and, in two cases, by
OpenCode/Codex under Claude Code's supervision — that is stated plainly in each
case study rather than implied otherwise. The one directly hands-on-keyboard
action found anywhere in this record is Hunter adding one permission line to his
own configuration himself (Case Study 2).

## Fresh verification runs for this package (2026-09-08, this session)

| Check | Result |
|---|---|
| `test-verify-repair.ps1` (verification controller's own adversarial suite) | 17 passed, 0 failed |
| Python-Inspector `pytest -q -rs` | 95 passed, 1 skipped, exit code 0, 153.23s |

Both match their historical baselines exactly — nothing regressed since the
underlying missions closed.

## Still open (Hunter's call, not decided here)

- Whether to schedule either Python-Inspector next-step candidate noted in Case
  Study 3 (secrets false-positive tuning, pip-audit lockfile support) — not
  scheduled, per this mission's boundary against silent scope expansion.
- Publication: when and how any of this leaves this folder. Not decided or
  actioned here.

## File map

```
Job-Portfolio-Local/
├── INDEX.md                        (this file)
├── NOT-FOR-PUBLICATION.md
├── interview-notes.md
├── 01-verification-system/case-study.md
├── 02-fleet-supervision/case-study.md
├── 03-python-inspector/case-study.md
└── 04-evidence-durability/case-study.md
```
