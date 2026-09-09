# Hunter Terry — AI Orchestration & Verification

I direct AI coding agents to build software and require evidence before accepting their work. My focus is defining clear tasks, setting permission boundaries, and making sure reported success matches what was actually checked.

**Seeking remote AI implementation, automation, or agent operations opportunities.**  
[GitHub profile](https://github.com/hunter-terry) · [Contact](mailto:hunterterry234@gmail.com)

## Selected work

These four case studies document software development and reliability work in my own environment.

| Project | Problem addressed | Documented result |
|---|---|---|
| [Verification system](01-verification-system/case-study.md) | Agents can claim success without valid checks. | A deterministic controller checks submissions against a frozen contract; 17 adversarial tests passed in the recorded run. |
| [AI-worker supervision](02-fleet-supervision/case-study.md) | Delegated agents can change unrelated files or report environment-specific failures. | Review caught an unauthorized test rewrite; separate execution distinguished sandbox failures from project failures. |
| [Python-Inspector](03-python-inspector/case-study.md) | Code scanning needs useful findings and controlled test execution. | Desktop inspection tool combining five checks and Docker test execution; latest recorded suite: 105 passed, 1 skipped. |
| [Evidence-durability repair](04-evidence-durability/case-study.md) | Routine reinstalls removed verification records. | Evidence storage was relocated and checked with per-file hashes across a real reinstall. |

## My contribution

I define the goals and acceptance criteria, choose the tasks and tools, make permission and scope decisions, and review reported outcomes before accepting work.

Claude Code and delegated OpenCode/Codex agents performed implementation and technical checking. The case studies distinguish my direction from agent execution. Separate review sessions and deterministic checks are part of the workflow; they are not claims of an external audit or of code I personally wrote.

## Project previews

**Verification controller** — a disposable demonstration of a frozen contract and a checked result.

![Verification controller console demonstration](01-verification-system/screenshots/verify-repair-console.png)

**Python-Inspector** — results from a synthetic fixture containing deliberately planted issues.

![Python-Inspector results screen](03-python-inspector/screenshots/results-screen.png)

## Validation and evidence

Recorded on **September 8, 2026**:

| Check | Recorded outcome | Qualification |
|---|---|---|
| Controller adversarial suite | 17 passed, 0 failed | Disposable test fixtures; not proof against an attacker with the same account permissions. |
| Python-Inspector `pytest -q -rs` | 105 passed, 1 skipped; exit 0 | One Docker-daemon-unavailable skip. Separate GUI checks also had four recorded environment-related failures. |

This repository publishes case studies and screenshots. Implementation repositories, raw transcripts, and detailed execution records remain local or private; their paths and commit identifiers are provenance references, not publicly reproducible evidence. Test results are historical records and were not rerun for this documentation update.

Each case study includes the problem, my role, the implementation, verification results, and limitations.

## Tools used

Claude Code · OpenCode with OpenRouter models · Codex · Python · PowerShell · Docker · Git
