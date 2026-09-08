# Job Portfolio — Local Package (prepared 2026-09-08)

**Status: published publicly on GitHub on 2026-09-08, at Hunter's explicit
direction, after local review. See [PUBLICATION-STATUS.md](PUBLICATION-STATUS.md).**

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

## Fresh verification runs for this package (2026-09-08)

| Check | Result |
|---|---|
| `test-verify-repair.ps1` (verification controller's own adversarial suite) | 17 passed, 0 failed |
| Python-Inspector `pytest -q -rs` | 105 passed, 1 skipped, exit code 0, 177.86s |

The verification controller's suite matches its historical baseline exactly.
Python-Inspector's suite count grew from 95 to 98 (secrets false-positive fix)
to 105 (Poetry/Pipenv dependency coverage, plus a follow-up correctness fix
found by an independent review of that same change) — all added coverage, no
regressions; the 1 skip is the same pre-existing, unrelated
Docker-daemon-unavailable skip throughout.

## Still open (Hunter's call, not decided here)

Both Python-Inspector next-step candidates noted in Case Study 3 (secrets
false-positive tuning, pip-audit lockfile support for Poetry/Pipenv projects)
are now completed — see Case Study 3 for both fixes and, for the Poetry/Pipenv
one, the independent second-look review that found and closed a real follow-on
defect in the first fix. Nothing is currently queued as an open candidate for
this package.

## File map

```
Job-Portfolio-Local/
├── README.md                       (GitHub landing page — same content as this file)
├── INDEX.md                        (this file)
├── PUBLICATION-STATUS.md
├── interview-notes.md
├── 01-verification-system/case-study.md
├── 02-fleet-supervision/case-study.md
├── 03-python-inspector/case-study.md
└── 04-evidence-durability/case-study.md
```
