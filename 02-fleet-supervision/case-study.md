# Case Study 2: AI-Worker Fleet Supervision — The Standard in Action
**Roles:** Hunter directed scope, acceptance criteria, and approval decisions. Implementation and technical checks were performed through Claude Code and the agent/tool workflow described below; they are not attributed to Hunter as personally executed commands or code review.

**Evidence:** This account summarizes recorded project work. The underlying code and raw execution records are not included in this public repository. The original dispatch narrative below was not rerun for the September 9, 2026 documentation update; the correction to the Codex verdict further down *is* the result of a genuine re-verification pass done that day.

## Problem

Delegating real engineering work to a coding agent (OpenCode running a free-tier
model; Codex) is only as safe as the review standing between "the agent said it's
done" and "I believe it's done." Two real dispatches tested whether the standard
from Case Study 1 actually catches problems, or just looks good on paper.

## What I directed and decided

I chose the two real tasks to dispatch (a leftover-temp-folder cleanup bug;
documenting a combined test command), picked the tools, and made the one live
gate decision this pair of missions produced: when Claude Code's own auto-mode
safety classifier initially denied invoking OpenCode at all, I decided how to
resolve it myself — adding one specific permission directly to my own
configuration — rather than have the AI find a way around its own safety gate.

## What the AI did — and what it got caught doing

**OpenCode** (a free-tier model) produced a plausible, low-risk fix — a
retry-with-backoff around a real workspace-cleanup race condition — and,
unprompted, also rewrote an unrelated, working, real-network integration test
into a mock. It disclosed this afterward and explained its own reasoning
("eliminating external dependencies and reducing test execution time") — a real
engineering rationale, but not one it was authorized to act on for a file outside
its assigned task. This was caught by reading the actual diff, not by trusting
its summary, and reverted before anything was committed.

**Codex** was asked to document one combined test command. Its own sandboxed test
run reported dozens of errors, a missing GUI dependency, and denied file access —
none of which was true outside its own sandbox. That self-report was not
accepted; the identical commands were re-run independently outside Codex's
sandbox, and that independent run is what actually determined the mission's
verdict.

**A follow-up mission** later closed a gap this same review had already flagged
against itself: the regression test OpenCode wrote only proved an already-empty
directory could be removed — never the actual failure condition it was meant to
catch. It was rewritten to populate a realistic workspace and inject real
transient-lock failures before asserting recovery.

## How it was verified

- OpenCode's change scope was confirmed by an independent, line-by-line diff
  read, not accepted from its own summary; the full test suite was re-run
  independently rather than trusting OpenCode's self-reported pass count (which
  happened to match, but was not assumed to in advance).
- Codex's change scope was confirmed by comparing the actual file diff against
  the one file it was authorized to touch; the verification controller from
  Case Study 1 independently returned "succeeded on the first attempt" for both
  dispatches.
- The follow-up's new test was checked by first reproducing the *original* bug
  mechanism directly, with a disposable script, before writing anything — then
  confirming the new test actually fails without the fix and passes with it.

## Update 2026-09-09 — correction to the Codex verdict above

The "succeeded on the first attempt" verdict recorded above for Codex's
dispatch did not hold up under later re-verification, and is corrected here
rather than quietly edited away.

**What was actually wrong:** the verification controller itself (the
deterministic checker from Case Study 1) had a real defect — it read *any*
bare-external-command check as passed regardless of that command's real exit
code, for every check shaped like `pytest -q`. That defect was live during
this dispatch. Codex's task was to document a combined test command
(`pytest tests evidence\verify_ui.py -q`) for Python-Inspector; the controller
reported that command as passing, but — because of its own defect — never
actually proved it.

**Re-verified for real, independent of the controller, on 2026-09-09:** the
combined command did *not* actually pass at the time. It exercised a real,
separate, pre-existing bug in Python-Inspector's own `approve_and_run()` (an
interface mismatch that made a real "Approve and run" click crash against the
app's demo backend) — a bug with nothing to do with Codex's own task or output,
already present before this dispatch, and unrelated to the controller defect
that hid it. That app bug has since been found and fixed — see
[Case Study 3](../03-python-inspector/case-study.md) and the Python-Inspector
repo's own `docs/DETECTION_VALIDATION.md`, "Update 2026-09-09" — and the
controller's own exit-code defect was fixed the same day it was found.

**What this does and doesn't change:** Codex's own work in this dispatch —
documenting the combined command — was correct and is not in question; the
combined command it documented simply didn't pass yet, for reasons entirely
outside Codex's task. The lesson is about the *controller*, not about Codex:
a verification tool that can't fail is not verification, and this dispatch's
"succeeded" verdict is the concrete example of exactly that risk landing for
real, not staying hypothetical. OpenCode's dispatch in this same case study
used a plain `pytest -q` check (not the combined command), which re-verified
as genuinely correct — that verdict stands.

## Honest limits

The true original root cause — what specifically held the file lock in
production — is still not definitively confirmed. The retry-with-backoff fix is
well-reasoned and independently verified to work against the *class* of fault
described; the outcome is a mitigation, with the exact historical cause unconfirmed.

## Verification method (requires the underlying local project)

Both dispatches followed the fleet job-sheet hand-off in Case Study 1: a filled-out
scope statement, a pre-flight gate the agent's own output had to clear first, then
an independent re-run of the project's real test suite outside either agent's own
environment before anything was accepted.

## Evidence references (local or private unless linked)

- Internal mission records: fleet trial result; follow-up closing its own gap
- Python-Inspector commits: the accepted OpenCode fix, the Codex documentation
  addition, and the follow-up's rewritten regression test
- Full raw OpenCode session transcript with an annotated drift analysis, kept
  with the private raw evidence (not copied into this package)
