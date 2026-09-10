# Case Study 3: Python-Inspector — Built, Hardened, and Validated Software
**Roles:** Hunter directed scope, acceptance criteria, and approval decisions. Implementation and technical checks were performed through Claude Code and the agent/tool workflow described below; they are not attributed to Hunter as personally executed commands or code review.

**Source:** [github.com/hunter-terry/python-inspector](https://github.com/hunter-terry/python-inspector) — full code, commit history, and `docs/DETECTION_VALIDATION.md`.

**Evidence:** This account summarizes recorded project work. Raw findings and file paths from the three private local projects used for detection-quality validation (see below) are kept private, not published — only the aggregate results are cited here and in the linked repo's docs. The pre-September-9 results below (container execution, repaint fix, detection-quality validation, the two earlier fixes) were not rerun for this update; the suite counts in "How it was verified" were re-run fresh on September 9, 2026 and reflect three changes made that day — a secrets-dedup fix, one app bug, and one flaky test (see the linked repo's `docs/DETECTION_VALIDATION.md` for detail).

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

- **Container execution and cancellation passed the recorded checks.** The hardened container
  (`--cap-drop ALL`, `--read-only`, a size-capped tmpfs) ran a real third-party
  project's full test suite clean (185 tests passed) and Python-Inspector's own
  suite; a real hung process inside the container was killed in about 2 seconds
  on cancellation, with no orphaned container left behind afterward.
- **A reproduced GUI repaint bug fixed, and save-error handling verified.** A repaint bug
  where the results screen's header rendered blank after a minimize/restore
  cycle (fixed with a targeted redraw trigger); and confirmation that a real
  save-permission failure — triggered through an actual file lock from another
  process, not a mock — renders the app's own error banner correctly.
- **Real-world detection-quality validation.** Ran the tool against three of my
  own real local projects. It caught a genuine, currently-published CVE in one
  project's pinned test dependency. It also surfaced two real product limits,
  documented rather than hidden: secrets detection had a 0-for-27 real-hit rate
  on "possible" findings in this sample (expected heuristic noise, not a defect,
  but worth discounting by default) — **follow-up changes described below** — and
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
  whether it actually decodes to valid JSON before being flagged — a filter whose broader effect on missed secrets has not been established by the recorded sample. Re-verified with a
  second live re-scan of one of the same three real projects — specifically
  the one that had contributed 25 of the original 27 findings, making it the
  richest source to re-check: **25 → 15 findings** on that one project,
  with the one deliberate real-secret test fixture in it still caught
  correctly.
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
- Fresh re-run, 2026-09-09, commit `f36a4af`: `pytest -q` → **107 passed, 1
  skipped, in 174.16s, exit code 0** — the count grew from 95 (before any fix)
  through 98 (secrets false-positive fix), 105 (Poetry/Pipenv coverage plus the
  independent review's own follow-up correctness fix), 106 (secrets-dedup fix,
  drafted by OpenCode and independently verified by Claude Code), to 107 (one
  more regression test added for a real interface bug found the same day, see
  below); the 1 skip at that point was a pre-existing, unrelated
  Docker-daemon-unavailable skip (closed later the same day — see below).
- Same commit, the combined suite (`pytest tests evidence\verify_ui.py -q -rs`,
  which adds `evidence/verify_ui.py` — the GUI regression file `pyproject.toml`
  excludes from the bare command above): **120 passed, 1 skipped, 1 failed, in
  238.33s**. The 1 skip and the 1 failure were the same root cause reported by
  two different tests — Docker Desktop's daemon could not be started in this
  environment during that pass — not a code regression; no other failures were
  observed. This supersedes an earlier claim that the GUI suite had multiple
  Docker-related failures: re-verification on 2026-09-09 found that claim was
  wrong about the cause for at least two of them (see below), which were a real
  app bug and a flaky test, not Docker.
- **Docker gap closed, same day, commit `7335eef`**: once Docker Desktop was
  restarted and confirmed reachable (`docker info`), the bare suite went to
  **108 passed, 0 skipped, 186.14s** and the combined suite to **122 passed, 0
  skipped, 0 failed, 247.00s**. The two tests previously blocked by Docker's
  unavailability —
  `test_live_isolated_run_executes_pytest_with_network_disabled` (previously
  skipped) and `test_real_scan_large_report_clipboard_and_docker_output`
  (previously failed) — were independently re-run by name and both confirmed
  passing (`2 passed in 128.64s`). No environment gap remains in either suite.
- **One real app bug and one flaky test, both found and fixed during that same
  re-verification pass** (by Claude Code, independent of any AI-fleet
  dispatch): (1) `approve_and_run()`
  always calls the backend with an `is_cancelled` argument that the documented
  interface and the demo `MockBackend` didn't accept — every real "Approve and
  run" click against the shipped demo app raised a `TypeError`, which had been
  masquerading as GUI-test flakiness; fixed by extending the interface and
  `MockBackend` to match `RealBackend`'s existing, already-correct signature,
  with a dedicated regression test proven to fail pre-fix and pass post-fix.
  (2) A GUI regression test for a minimize/restore repaint nudge polled too
  coarsely (~20ms) to reliably observe a ~1ms state change, roughly a 1-in-4
  real failure rate with no underlying app defect; fixed by tracing the actual
  repaint call instead of racing a polling loop, 10/10 clean runs after. Full
  detail, commits, and exact commands: the linked repo's
  `docs/DETECTION_VALIDATION.md`, "Update 2026-09-09."
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
statistical claim about the tool's overall accuracy.

## Verification method

From the project root, with its virtual environment active:
`pytest -q -rs` runs the full suite. The three real-world validation targets
were external projects private to Hunter and are not published; the method
and aggregate per-run results are in the public repo's own
`docs/DETECTION_VALIDATION.md`.

## Evidence references

- Repo: [github.com/hunter-terry/python-inspector](https://github.com/hunter-terry/python-inspector)
- Commits: the repaint fix; the detection-quality validation pass; the
  [secrets false-positive fix](https://github.com/hunter-terry/python-inspector/commit/6a7b0c1);
  the [Poetry/Pipenv dependency-coverage fix](https://github.com/hunter-terry/python-inspector/commit/2f21eeb)
  and its [independent review's follow-up correctness fix](https://github.com/hunter-terry/python-inspector/commit/3dc4985);
  the [secrets-dedup fix](https://github.com/hunter-terry/python-inspector/commit/4079e05);
  the [approve_and_run() interface fix and repaint-test deflake](https://github.com/hunter-terry/python-inspector/commit/f36a4af)
- Internal mission records (private): Docker/GUI verification; detection-quality
  validation; secrets false-positive fix; Poetry/Pipenv coverage fix and its
  independent second-look review
- `docs/DETECTION_VALIDATION.md` in the linked repo; the underlying `evidence/`
  files for the three private validation projects are kept local, not published
