---
name: diamond-qc
description: >-
  Adversarial multi-agent quality control for any artifact with checkable claims
  (study notes, code, prose, research, configs, data). The author drafts, two
  decorrelated auditors hunt for errors from different lenses, and a meta-auditor
  audits the auditors. Verifies by execution and evidence, not assertion. Use when
  the user says "Diamond QC", "the Diamond", "who watches the watchmen", or asks to
  rigorously verify, fact-check, or QC something before it ships, especially when
  it is going to other people, contains code or numbers, or is hard to reverse.
---

# The Diamond QC

A protocol for catching errors in an artifact before it ships. Named for the
shape: one author at the top, two independent auditors across the middle, one
meta-auditor at the bottom point. The bottom point is the "who watches the
watchmen" step.

It exists because a single reviewer (including the author) shares blind spots
with itself. Independent, differently-aimed verification catches what one pass
cannot, and a meta layer catches what the verifiers themselves miss or get wrong.

---

## When to use it (tiering)

Match effort to risk. Do not run the full diamond reflexively; it costs roughly
3 to 5 subagents per item.

- **SKIP** — trivial, easily reversible, or single-fact work, and anything you
  have already verified by execution. Just do it. Do not invoke the diamond.
- **LIGHT** — low to moderate stakes with checkable claims (a small code change,
  a short factual answer). Author drafts and self-verifies by running it, then
  **one** independent auditor checks by execution. No meta layer. Then
  verify-the-fix.
- **FULL** — high stakes: shipped to other people, contains statistics or
  safety-relevant logic, is hard to reverse, or the user asked for the Diamond by
  name. Author drafts and self-verifies, **two decorrelated auditors** review,
  **one meta-auditor** audits them, then verify-the-fix.

Escalate by **risk, not repetition**. A second identical pass is correlated with
the first and mostly re-confirms it. If you need more assurance, add a *different*
lens, not another copy of the same one.

When unsure which tier, state the tier you picked and why in one line, then
proceed.

---

## The roles

**Author (usually you, the main agent).** Produce the artifact. Before handing it
off, verify every claim you can: run every code block, compute every number,
trace every fact to its source. Fix the malformed and the obviously wrong
yourself. Do not pass known-shaky work to the auditors to find.

**Auditor A — fidelity / correctness against ground truth.** Does every claim
match the source, spec, or reality? Catch fabrications, distortions,
misattributions, and unsupported assertions. Classify each claim: supported /
added-but-true / wrong. Quote exact text for each finding.

**Auditor B — technical / empirical, by execution.** Run the code. Run the
numbers. Check the types, shapes, boundaries, edge cases. Report the actual
output next to the claimed output. "I checked it" is not allowed; show the run.

**Meta-auditor — audits the auditors.** Two symmetric jobs, both required:
1. Catch what **both** auditors missed (their shared blind spot).
2. **Overturn** what an auditor got wrong (false positives and false alarms).
Spot-check the auditors' key claims independently. Do not trust their summaries.
Then deliver the final verdict and the minimal set of fixes.

### Decorrelation is the whole point

The two auditors must aim at the artifact from **different angles**. Same artifact,
different lens (fidelity vs execution; or, for risky domains, distinct adversarial
framings). Two auditors running the same check share the same blind spot and waste
the second seat. Diversity of attack, not redundancy, is what catches errors.

For high-risk domains (statistics, security, anything subtle), frame auditors
**adversarially**: their job is to *try to refute* each claim and default to
skeptical, concluding "clean" only after a genuine attempt to break it.

---

## Core rules

1. **Verify by evidence, not assertion.** Every numeric or code claim gets run.
   Every factual claim gets traced to a source. Recall is not verification.
2. **Decorrelate the auditors.** Different lenses, never the same check twice.
3. **The meta-auditor cuts both ways.** It promotes real misses *and* kills bad
   findings. A meta layer that only rubber-stamps is half-built.
4. **Correctness is necessary but not sufficient.** Every fix must also pass a
   fit-to-purpose gate: does this actually serve the end user and the context? A
   technically correct change that contradicts the user's situation (their grader,
   their platform, their constraints) is still wrong. When correct-but-unfit,
   prefer keeping the correct content *plus* a transparent caveat over silently
   overriding or silently complying.
5. **Run once per tier, escalate by risk.** No identical second pass.
6. **Verify-the-fix.** Any change the QC triggers gets re-checked against the
   diff before it ships. Do not let an unverified fix out the door.
7. **No silent caps.** If you bounded coverage (sampled, skipped, capped), say so.

---

## How to run the FULL diamond

1. **Draft + self-verify.** Produce the artifact; run/trace everything you can.
2. **Spawn both auditors in parallel** (one message, two Agent calls, so they run
   concurrently). Give each: the artifact, the ground truth (file paths, spec,
   data), its specific lens, and the instruction to lead with errors and to state
   clearly if none survive a genuine attempt to break it. Use general-purpose
   subagents (they can read and run code); Explore is fine for read-only fidelity
   checks.
3. **Read both reports.** Note real findings, contested findings, and gaps.
4. **Spawn the meta-auditor**, handing it the artifact, the ground truth, and
   **both auditor reports**. Tell it explicitly: spot-check independently, find the
   shared blind spot, overturn any wrong finding, rule on correct-but-unfit fixes,
   and return a final verdict with the minimal fix list.
5. **Apply the minimal fixes.** Then **verify-the-fix** (diff-only re-check; run
   any changed code).
6. **Report** what each layer caught, what was fixed, and residual risk. Be honest
   about confidence and about anything left unverified.

For the LIGHT tier, do steps 1, 2 (one auditor), 5, 6.

---

## Reporting format

Each auditor leads with **outright errors** (or a clear "none after genuine
attempts"), then lesser findings, with exact quotes and, for Auditor B, actual
command output. The meta-auditor returns: confirmed findings, overturned
findings, anything both missed (with severity), rulings on any correct-but-unfit
fixes, and a final **publish-ready?** verdict with the minimal fixes only.

End the whole run with a short scorecard: what each layer caught, the fixes
applied, and a calibrated confidence statement.

---

## Worked example (the origin run)

Built five interwoven study notes for a course, one Diamond per note. The pattern
held and each layer earned its seat:
- An author-added connective claim that no source supported (meta caught it; two
  auditors had waved it through).
- An auditor false positive where flagged wording was actually faithful to source
  (meta overturned it).
- A quiz-safety hazard: a *correct* fix would have set the student up to fail a
  course-keyed quiz, so the fix kept the correction plus an exam-safety caveat
  (the fit-to-purpose gate, rule 4).
- A statistics miscalibration (a confidence interval that only achieved its stated
  coverage at large sample size), found by an empirical coverage simulation.
- A version-drift claim about library defaults, caught by running it on the actual
  current version.

The lesson worth carrying: the highest-value catches were not raw factual errors.
They were the connective tissue the author invented, the verifier's own
overreach, and correct-but-unfit fixes. Aim the diamond there.
