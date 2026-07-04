# PRD — The Fusion Engine (V1)

**Status:** Final for V1. Reconciled against CLAUDE.md, the frozen build authority. Where this document and CLAUDE.md differ, CLAUDE.md wins.
**Scope:** the brain. A single evaluation pipeline, one box, transparent rules.

---

## 1. What it is

The fusion engine is a cross-referencing baseline deviation detector. It is not a diagnostic medical AI. It merges the physical stream (wearable vitals) with the cognitive stream (June's flags), reads both against the resident's own personal baseline, and when a pattern crosses a threshold, packages the deviation as raw evidence for a human to judge.

Every output ends with the frozen line: **"— not a diagnosis. Nurse judgment required."**

## 2. Why this design

- **For the caregiver:** defeats alarm fatigue. Alerts are rare, cross-referenced, and arrive carrying their own proof.
- **For the elder:** respects chronic reality. A COPD resident's low oxygen is his normal, so it never fires on him. Deviation is measured from the person, not the population.
- **For the ecosystem:** avoids the regulatory trap. By outputting evidence and soft hypotheses rather than diagnoses, the system remains decision support rather than a regulated diagnostic device. The transparent ruleset means every alert can explain exactly why it exists.

## 3. Architecture: the four-stage loop

**A — Dual receivers.** The engine generates no data. It reads:
- The physical stream (vitalsSamples): heart rate, SpO2, sleep/wake state.
- The cognitive stream (cognitiveFlags): June's structured observations, with the elder's exact quotes.
- Sleep history (nightSummaries), which the infection rule depends on.

**B — The baseline ledger.** Before judging anything, it loads the resident's personal normal from the residents collection: resting heart rate, SpO2, sleep hours and interruptions, and whether repetition is normal for this person. In V1 baselines are seeded reference values, written by hand. Learning baselines from observed data is Phase 2, and the product never claims otherwise.

**C — The logic core.** A transparent, hand-written ruleset (lib/rules.ts) examining the last 3 days of flags, the last day of vitals, and the last 3 nights of sleep against the baseline. The full V1 ruleset, frozen in CLAUDE.md:

- **R1 — spoken emergency:** any distress flag. Can't breathe, fall mention, or chest pain go to S4; otherwise S3.
- **R2 — oxygen drop:** three consecutive readings below normal minus 3 gives S3. Any reading below 90 and below normal minus 4 gives S4.
- **R3 — heart racing:** three consecutive readings above 1.3x normal gives S3.
- **R4 — the infection pattern (the headline):** within 3 days, a cognitive flag (respecting repetitionNormal) plus 2+ bad nights plus average heart rate up 10% gives S2. No single stream catches this; together they do.
- **R5 — sinking mood:** 3+ loneliness or hopelessness flags in a week gives S2.
- **R6 — dizzy plus racing:** a dizziness complaint plus heart-rate drift within 6 hours gives S2.
- **R7 — equipment:** low battery or band out of range gives S1 on the ops channel, never mixed with clinical alerts.
- **R8 — silence:** no conversation and no change in body readings for 12 hours gives an S2 wellness check. "No data" refuses to mean "fine."
- **R9 — the silent emergency:** alarming vitals plus an unresponsive elder gives S4. The collapsed person can't talk, so the silence plus the vitals together are the proof.

**D — The output generator.** A triggered rule writes a TriageEvent:
- Severity on the frozen S0–S4 scale.
- A soft-language title and hypothesis, ending in the frozen disclaimer.
- An evidence list citing the exact flags, vitals, and sleep records that fired it, including the elder's own words.

## 4. The no-spam law (never cut)

- Before creating any alert, check for an open alert for the same resident and same rule. If one exists, merge the new evidence into it and bump its timestamp. A second copy is never created.
- After a nurse resolves or dismisses an alert, that rule stays quiet for that resident for 6 hours.
- The acceptance test: the UTI scenario must produce exactly one alert card that accumulates evidence across two days.

This mechanism, not the rules themselves, is the answer to alarm fatigue.

## 5. Severity (frozen; S4 is the emergency)

| Level | Name | Meaning | Nurse sees |
|---|---|---|---|
| S0 | Ambient | normal, just learning | nothing; stored only |
| S1 | Note | one mild thing | daily report only; equipment goes to the ops tab |
| S2 | Review | a cross-signal pattern, not urgent | a silent card in the queue |
| S3 | Check soon | check within 15 minutes | card pinned to top, plus a chime |
| S4 | Now | emergency | full-width red banner, tone until acknowledged |

Rules may be tuned. Meanings may not.

## 6. Phase 2: the transplant

V1 deliberately writes perfectly structured longitudinal data: every flag, vital, night, and outcome, keyed by resident. In Phase 2 the hand-written logic core is replaced by an anomaly-detection model trained on each resident's accumulated history. Same inputs, same outputs, so the swap dismantles nothing. The rulebook is the substrate that makes its own successor trainable.

## 7. Known blindspots (stated, not hidden)

- Blind to localized issues (a rash, a bruise) unless the elder mentions them to June.
- Blind to the body when the wearable is off; relies on the cognitive stream alone until compliance is restored.
- Thresholds are informed guesses pending clinical validation, which is why every alert defers, in its final line, to nurse judgment.
