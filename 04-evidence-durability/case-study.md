# Case Study 4: Evidence-Durability Repair — Found, Fixed, Hash-Verified

## Problem

The verification controller from Case Study 1 is only as trustworthy as its own
evidence trail. Its default evidence folder turned out to live *inside* the exact
directory tree that the skill installer deletes and recreates on every reinstall
— so the proof that past repairs were actually verified was being silently
destroyed by routine maintenance on the very tool that relies on it.

## What I directed and decided

I reviewed two possible fixes and chose to relocate the evidence root to a
location the installer structurally never touches, rather than changing the
installer's own reinstall behavior — its "delete and recreate cleanly" behavior
was already correct; weakening it would have worked around a problem that
actually belonged in the other file.

## What was found and fixed

Root cause, confirmed by reading both scripts directly rather than assumed: the
controller's old default evidence path resolved to a subfolder of the exact
directory the installer's own cleanup step deletes before recopying. Fixed by
pointing the controller's default evidence root at a new location that sits
alongside, not inside, the installed skill tree — matching an existing
sibling-folder precedent already used for the installer's own backups.

## How it was verified

- Migrated all existing evidence folders to the new location and confirmed every
  file byte-identical (SHA256, per file) before deleting any original.
- Ran a live canary mission end-to-end from the installed controller copy at the
  new default path, then ran a real second reinstall — and hash-confirmed the
  canary's evidence was still byte-identical afterward, proving the fix under the
  exact failure condition it was meant to fix, not just in theory.
- Re-ran the controller's full adversarial test suite before and after: all
  passed, both times.

## Honest limits

This closes the specific failure mode found (evidence loss on reinstall). It does
not change the same-user trust boundary already disclosed in Case Study 1 — this
is a durability fix, not a new security boundary.

## Reproduction

The evidence index file at the controller's canonical evidence root lists every
completed mission and its verdict — including two entries written before and
after this fix's own verification reinstall, still present and unchanged.

## Evidence

- Internal mission record: evidence-durability repair result
- Two commits in the maintenance-skills repo (evidence-root relocation)
- The live evidence index file itself, now proven to survive a reinstall
