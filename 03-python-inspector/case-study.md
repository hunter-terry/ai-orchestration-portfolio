# Case Study 3: Python-Inspector — Built, Hardened, and Validated Software

## Problem

Python-Inspector is a desktop tool that scans a local folder or a GitHub
repository with five independent tools (ruff, bandit, pip-audit, detect-secrets,
plus a first-party repo-config checker) and can run a project's own test suite
inside a hardened, disposable Docker container. Two questions matter for a tool
like this: does the sandbox actually hold under real load, and can its findings
actually be trusted against real code — not just its own test fixtures?

## What I directed and decided

I set those two questions as the actual bar — not "does it run without
crashing" — picked the real targets used to answer them (a real GitHub project;
three of my own real local projects, unrelated to Python-Inspector itself), and
decided which of the previously reported "rough edges" were worth a live
click-through investigation versus which could wait.

## What was found and fixed

- **The sandbox holds under real load.** The hardened container
  (`--cap-drop ALL`, `--read-only`, a size-capped tmpfs) ran a real third-party
  project's full test suite clean (185 tests passed) and Python-Inspector's own
  suite; a real hung process inside the container was killed in about 2 seconds
  on cancellation, with no orphaned container left behind afterward.
- **Two real, live-reproduced GUI bugs, fixed and re-verified.** A repaint bug
  where the results screen's header rendered blank after a minimize/restore
  cycle (fixed with a targeted redraw trigger); and confirmation that a real
  save-permission failure — triggered through an actual file lock from another
  process, not a mock — renders the app's own error banner correctly.
- **Real-world detection-quality validation.** Ran the tool against three of my
  own real local projects. It caught a genuine, currently-published CVE in one
  project's pinned test dependency. It also surfaced two real product limits,
  documented rather than hidden: secrets detection had a 0-for-27 real-hit rate
  on "possible" findings in this sample (expected heuristic noise, not a defect,
  but worth discounting by default) — **since fixed, see below** — and
  dependency-vulnerability scanning has no coverage at all for a project that
  uses Poetry or Pipenv instead of a `requirements.txt` — **since fixed, see
  below**.
- **Secrets false-positive fix, verified live.** The 0-for-27 noise above
  traced to two root causes: `detect-secrets --all-files` scanning into
  tool-cache and dependency directories (`.venv`, `.pytest_cache`,
  `.ruff_cache` — 76 of 83 findings when run against this repo's own tree),
  and its `Base64 High Entropy String` plugin flagging non-secret,
  JSON-encoded log payloads as possible secrets. Fixed both: tool-cache
  directories are now excluded by default (reusing the project's existing
  ignore-list, not a new mechanism), and a base64 hit is now checked against
  whether it actually decodes to valid JSON before being flagged — narrow by
  construction, so it does not risk hiding a real secret. Re-verified with a
  second live re-scan of the same class of real project: **25 → 15
  findings**, with the one deliberate real-secret test fixture in that
  project still caught correctly.
- **Poetry/Pipenv dependency coverage, added and then independently
  corrected.** `pip-audit` originally read only `requirements.txt`-family
  files, so a project declaring dependencies via `pyproject.toml` (Poetry or
  PEP 621) or a Pipenv `Pipfile`/`Pipfile.lock` got zero
  dependency-vulnerability coverage — a real, silent blind spot, not
  discovered as a hypothetical: it's the same class of file most modern
  Python projects actually use. Fixed by extracting exactly-pinned
  dependencies from whichever manifest is present and auditing them through
  the same `pip-audit` invocation already used for `requirements.txt`; two
  new fixtures confirmed real findings where the tool previously reported
  `Unavailable`. A **separate, independent review session** — deliberately
  run with no memory of how the fix was built, specifically to avoid
  rubber-stamping the first session's own report — then found a real
  follow-on defect the original fix missed: a PEP 508 extras marker (e.g.
  `requests[security]==2.25.0`, a common real-world pattern) contained its
  own bracket pair, which broke the array-parsing regex and silently
  dropped every pin in that array, not just the one with extras. It failed
  safe (degraded to `Unavailable`, never a false "audited, clean" claim),
  but it was a real coverage loss for a very ordinary dependency shape.
  Fixed with a small bracket-depth scanner in a follow-up commit, verified
  through the same bounded pass/fail verification contract used elsewhere in
  this package, independently of the original fix's own author.

## How it was verified

- Docker/GUI verification: real container runs against a real GitHub project and
  Python-Inspector's own repo, a real cancellation test, and a live manual
  click-through of every screen — including a genuine save error deliberately
  triggered through a file-sharing violation, the one way to reach that code
  path, since the OS's own save dialog pre-checks and blocks the simpler
  permission-denied case before the app ever sees it.
- Detection-quality validation: every "strong" and "possible" finding used as
  evidence above was hand-checked against the actual flagged source line, not
  accepted from the tool's own confidence label.
- Fresh re-run, 2026-09-08: `pytest -q -rs` → **105 passed, 1 skipped, in
  177.86s, exit code 0** — the count grew from 95 (before either fix) to 98
  (secrets false-positive fix) to 105 (Poetry/Pipenv coverage plus the
  independent review's own follow-up correctness fix); the 1 skip is
  confirmed the same pre-existing, unrelated Docker-daemon-unavailable skip
  throughout.
- The secrets false-positive fix was also checked against `evidence/verify_ui.py`
  (the separate GUI regression suite): 4 pre-existing failures (all citing
  Docker Desktop's engine not running) reproduce identically on the fix and on
  the unmodified prior commit via `git stash`, confirming the fix caused no
  new GUI regression.
- The Poetry/Pipenv fix was independently re-verified end to end by a second,
  separate session with no memory of the first: full diff read line by line
  against the change's own claims, full suite re-run fresh, both new fixtures
  re-run live (not by reading code) confirming real findings with correct
  file paths and line numbers, and the existing `requirements.txt` path
  re-confirmed byte-for-byte unchanged.

![Python Inspector's real Results screen after scanning the repo's own synthetic "vulnerable_project" test fixture — 21 findings across 4 confidence tiers, including a confirmed shell=True subprocess issue and a possible SQL-injection pattern.](screenshots/results-screen.png)

The screenshot above is a real scan of the project's own known-answer test
fixture (`tests/fixtures/vulnerable_project` — a folder of deliberately planted,
synthetic bugs the project ships specifically for this kind of demonstration
and its own automated tests, not a real project or personal data). It shows the
actual confidence tiering in action: a `High`/`Confirmed failure` for a real
`subprocess` call with `shell=True`, next to a `Medium`/`Informational`
possible-SQL-injection pattern that needs a human read before acting on it —
exactly the distinction Case Study 3's detection-quality validation relies on
when separating confirmed findings from ones that need judgment.

## Honest limits

One previously-reported "rough edge" — a folder-picker crash — could not be
reproduced across 9 live attempts, and is recorded as "does not currently
reproduce," not proven structurally fixed. The CVE catch and the false-positive
rate above are one-sample observations against three real projects, not a
statistical claim about the tool's overall accuracy. The project has no GitHub
remote yet — a deliberate choice to stay local until it's ready for public
posting, not an oversight.

## Reproduction

From the project root, with its virtual environment active:
`pytest -q -rs` runs the full suite. The three real-world validation targets are
external projects not included in this package; the method and per-project
results table are in the project's own `docs/DETECTION_VALIDATION.md`.

## Evidence

- Commits: the repaint fix; the detection-quality validation commit and its
  6 evidence report files; the secrets false-positive fix (`bbbf504`); the
  Poetry/Pipenv dependency-coverage fix (`cae1816`) and its independent
  review's follow-up correctness fix (`b6e2ad8`)
- Internal mission records: Docker/GUI verification; detection-quality
  validation; secrets false-positive fix; Poetry/Pipenv coverage fix and its
  independent second-look review
- `docs/DETECTION_VALIDATION.md` and the `evidence/` folder in the Python-Inspector repo
