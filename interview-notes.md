# Interview Notes

Plain-language notes for each case study — problem, what I decided, what the AI
did, one real failure, the fix, how I checked it, and what's still limited.
Written to practice out loud, not to read verbatim.

## 1. The verification system

**Problem**: once AI agents are actually changing code, the biggest risk is
trusting their own claim that it worked.

**What I decided**: nothing gets marked done on my own record until something
other than the AI's own output checks it. I designed the shape of that check — a
small deterministic script, plus a written standard for tools outside Claude Code
entirely.

**What the AI built**: a controller that locks in a test contract before any fix
is attempted, then judges up to 3 attempts against it — never trusting a "tests
passed" claim at face value.

**One real failure it caught in itself**: a mission record once told me the wrong
thing about which piece of my own tracking needed fixing. It was wrong, and the
system's own logging discipline is what caught and corrected it on the record,
rather than letting it slide.

**How I checked it**: ran its own adversarial test suite fresh — 17 out of 17
pass, including scenarios like "a candidate forges its own success result" and "a
check crashes mid-run" — both handled correctly (rejected, or failed closed).

**Still limited**: those 17 tests run against disposable test fixtures, not a
live project. And I say plainly that this isn't tamper-proof against someone with
the same level of access as me on my own machine — it catches ordinary mistakes
and dishonest shortcuts, not a determined attacker with equal permissions.

## 2. AI-worker fleet supervision

**Problem**: is the review process above real, or does it just sound good on
paper?

**What I decided**: which two real bugs to hand to two different AI coding
tools, and — when my own safety classifier blocked the first attempt — how to
resolve that myself rather than have it worked around.

**What the AI did**: one tool fixed the assigned bug fine, but also quietly
rewrote an unrelated test into a mock without being asked. The other reported
test failures from inside its own sandbox that weren't real.

**One real failure, caught**: both of those — caught by actually reading the
diff and re-running the checks myself, not by trusting either tool's own
summary.

**How I checked it**: independent re-run of the full test suite outside each
tool's own environment; the file diff checked against exactly what was
authorized.

**Still limited**: the real original cause of the underlying bug still isn't
100% pinned down — I call the fix "mitigated," not "definitively solved."

## 3. Python-Inspector

**Problem**: does a security-scanning tool's sandbox actually hold, and can you
trust what it reports about real code?

**What I decided**: what counted as a real test — a real GitHub project, three
of my own real projects — instead of just its own toy test fixtures.

**What the AI built/found**: a Docker-hardened sandbox that survived a real
hung-process cancellation cleanly; two real GUI bugs found and fixed live; and,
running it for real, it caught an actual published CVE in one of my other
projects.

**One real failure, disclosed**: the tool's secrets scanner had zero real hits
out of 27 "possible" flags in this sample — not a bug, but worth knowing before
trusting that category blindly.

**How I checked it**: hand-verified every flagged finding against the actual
source line myself, rather than trusting the tool's own confidence label.

**Still limited**: one previously reported crash never reproduced again — I call
that "not currently reproducing," not "fixed for certain."

## 4. Evidence-durability repair

**Problem**: the tool from Case Study 1 was silently losing its own evidence
every time I reinstalled my skills.

**What I decided**: fix the evidence path, not the installer — the installer's
"wipe and recreate cleanly" behavior was already correct; the bug was the other
file assuming it wouldn't be.

**What the AI did**: relocated the evidence root outside the reinstalled folder.

**How I checked it**: hash-compared every migrated file before deleting the
original, then ran a real second reinstall and confirmed the evidence survived
byte-for-byte.

**Still limited**: this fixes the one failure mode found — it isn't a new
security boundary, just a durability fix.
