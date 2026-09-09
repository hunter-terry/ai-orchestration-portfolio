# Case Study 1: The Verification System — Keeping AI-Executed Work Honest
**Roles:** Hunter directed scope, acceptance criteria, and approval decisions. Implementation and technical checks were performed through Claude Code and the agent/tool workflow described below; they are not attributed to Hunter as personally executed commands or code review.

**Evidence:** This account summarizes recorded project work. The underlying code and raw execution records are not included in this public repository. Results below were not rerun for the September 9, 2026 documentation update.

## Problem

Once AI agents (Claude Code itself, and non-Claude coding agents like OpenCode and
Codex) are doing real, repository-modifying work, the central risk isn't that the
AI can't code — it's that its own claim of success can't be trusted by default. An
agent, or a tool it invokes, can report "tests pass" or "done" while the tests
didn't actually run, ran against stale code, or were quietly weakened to pass.
This is true even for Claude Code's own work, not just a third-party tool's.

## What I directed and decided

I set the standing rule: no AI-executed repair is reported "done" on my own record
until a mechanism independent of that AI's own output says so. I decided the shape
of that mechanism — a small, deterministic, local controller, not another model
and not a vibe check — with an explicit, honestly-scoped threat model, plus a
parallel written standard for delegating to tools outside Claude Code entirely.

## What was built (AI-executed, under that direction)

- **A verification controller**: freezes a contract (target, allowed write scope,
  protected paths, exact check commands, expected exit codes) *before* any repair
  edit is made, then judges each candidate submission against that frozen
  contract — capped at 3 submissions per mission. It recomputes every check
  itself; it never reads a "tests passed" claim from the repair and takes it at
  face value.
- **A fleet job-sheet standard**: the same principle extended to OpenCode and
  Codex, two coding agents with no access to that controller. A mandatory
  hand-off (exact scope, allowed files, done-criteria) plus a cheap mechanical
  pre-flight gate (the real check actually ran and exited clean; the file diff
  matches the allowed list exactly; capped at 3 tries) — and even after that gate
  passes, Claude Code independently re-runs the check and reads the actual diff
  itself before logging anything "Completed."
- **A mission-tracking contract**: a manual, never-automatic checkpoint lifecycle
  (Draft → Ready → In Progress → one of five honest terminal states) with a
  mandatory structure for every result record — verdict, timestamped history,
  authorization source, facts, findings with stable IDs, evidence, exact
  validation commands and exit codes, tested-vs-assumed, remaining risks,
  rollback, at most one open decision left for me — and a close-out self-check
  that re-scans the *entire* mission lane for drift every time, not just the
  current mission's own two records.

## A real failure this system caught in itself

One mission record initially told me a different mission's checkpoint needed a
manual fix. That claim was wrong, confirmed directly against the mission's own
subject matter, and the record says so plainly rather than quietly fixing it:
"Claude Code incorrectly told Hunter that a different Maintenance row... needed
its Status/Checkpoint manually synced to Completed. That was wrong... No
[tracking] write was made to that row; flagging only so the correction is on
record." The system's own discipline — log first, don't paper over — is what
surfaced and corrected this before it caused real bookkeeping drift.

## How it was verified

The recorded controller adversarial self-test run on 2026-09-08 returned: **17 passed, 0 failed**. It exercises real adversarial scenarios, not
just happy-path checks:

- known-good candidate passes; known-bad candidate fails; a repaired candidate
  passes on a later attempt
- an unchanged resubmission is rejected without consuming a real attempt
- a weakened frozen contract is rejected
- a forged or edited attempt result is rejected
- a controller-file tamper is detected; an arbitrary protected-path tamper is
  detected
- a 3rd still-failing submission exhausts and names the exact failing check; a
  4th attempt after exhaustion is refused, not silently run
- a timeout fails closed, not "succeeded"; a checker crash fails closed too
- a resumed mission continues at the correct attempt number, not duplicated
- the evidence index is created and correctly records the mission's outcome

It has also been exercised for real, twice, against actual OpenCode/Codex output
— see Case Study 2.

![The verify-repair.ps1 controller freezing and judging a disposable demonstration mission — real output, not a mockup.](screenshots/verify-repair-console.png)

The screenshot above is a real run against a disposable demo mission set up
solely to produce this screenshot (`portfolio-screenshot-demo-20260908`, target
a scratch folder, one trivial check) — not a real repair, and not one of the 17
adversarial tests. It shows the same real mechanics: `FROZEN` records both the
contract and controller digests before anything runs, then `RunAttempt` judges
the (trivial) check itself and prints `SUCCEEDED on attempt 1/3` — with
`protected paths: 1` confirming the controller protects itself automatically
even though this demo contract declared zero protected paths of its own.

## Honest limits

The 17 scenarios above run against disposable test fixtures, not a live
repository. The controller's own documentation states its threat model plainly:
it protects against an ordinary repair candidate lying about success, not against
a fully malicious process running under the same user account on the same
machine — a same-user trust boundary, explicitly never called "tamper-proof."
The mission-tracking self-check is bookkeeping discipline, not a technical
control.

## Verification method (requires the underlying local project)

From the maintenance-skills repo root: `powershell -File scripts\test-verify-repair.ps1`
runs the full adversarial suite against disposable `%TEMP%` fixtures — recorded as using disposable fixtures without touching a live repository.

## Evidence references (local or private unless linked)

- `scripts/verify-repair.ps1`, `scripts/test-verify-repair.ps1` (maintenance-skills repo)
- `references/verify-repair-controller.md`, `references/fleet-job-sheet.md`,
  `references/work-inbox-maintenance-contract.md` (same repo)
- Fresh run, 2026-09-08: 17 passed, 0 failed
- Two real production uses of the controller referenced in Case Study 2, both
  `SUCCEEDED on attempt 1/3`
