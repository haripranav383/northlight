# PRD — June, the Companion (V1)

**Status:** Final for V1. Reconciled against CLAUDE.md, the frozen build authority. Where this document and CLAUDE.md differ, CLAUDE.md wins.
**Scope:** the resident-facing surface. One page, one presence, voice-first.

---

## 1. What June is

June is an anti-app. She removes the friction of modern technology for a resident in assisted living: no menus, no icons, no navigation. A persistent, warm, voice-first window that behaves like a piece of smart, living furniture.

Her mission is double, and honest about it: she gives the elder consistent, non-judgmental companionship, and she serves as a passive clinical sensor for the care loop — noticing how the elder talks, never diagnosing what it means.

## 2. Problems she solves

- **For the elder:** replaces silence with presence, and removes the need to operate anything. June listens and waits.
- **For the caregiver:** removes charting burden by logging conversations and extracting structured observations automatically.
- **For the ecosystem:** captures the blind spot. The early cognitive signs — hesitation, repetition, mood shifts, complaints made in passing — that no wearable can see, delivered to the fusion engine as evidence.

## 3. Her role, precisely

- **June notices; the brain decides.** In every conversation she extracts notable moments into the fixed flag buckets (physical_complaint, cognitive, mood, sleep_complaint, appetite, distress), each with a subtype, the elder's exact quote, a severity hint, and a confidence score, per the frozen CognitiveFlag shape.
- She does not judge whether a flag matters, does not cross-reference vitals, and never sees or mentions body data. The fusion engine performs all interpretation. This separation keeps June simple and the brain swappable.
- She never mentions vitals, alerts, or the dashboard to the elder. She never uses diagnosis language.
- When a nurse's task arrives, she delivers it conversationally and warmly ("Sarah asked me to remind you...") and records the elder's confirmation or decline.

## 4. The interface (three layers, back to front)

- **The canvas (background).** A calm background with a gentle time-of-day tint (morning, day, evening, night) to help ground the resident in time. Low motion always; nothing that could cause vestibular fatigue. Live weather rendering is Phase 2.
- **The presence (center).** One organic element: the breathing glow. It breathes slowly when idle, glows amber while listening, and pulses softly while June speaks. It must feel biological, never mechanical. This is the signature animation of the product and the only prominent motion on the page.
- **The accessibility deck (bottom).** Massive, high-contrast subtitles of everything June says, and a live waveform line that activates only while the elder is speaking, so they can see they are being heard. Voice is primary; a large text input always works as the fallback. If the microphone fails, the demo does not.

## 5. Distress, honestly scoped

- If an elder tells June something alarming ("I can't breathe," "I fell," chest pain), the flag she writes carries it to the brain instantly, and rule R1 raises the top-level alert.
- If an elder goes silent while their vitals look wrong, rule R9 raises the alarm. The silence itself is the evidence.
- That is the honest extent of V1's emergency handling, and the boundary is stated plainly wherever the product speaks: **Northlight is not a fall alarm or nurse-call replacement and does not attempt to catch acute events in real time.**

## 6. Technical requirements

- Kiosk-style single page at /companion/[residentId].
- Browser speech recognition and synthesis, with the text fallback always available.
- Conversation memory per the frozen CompanionSession and companionMemory shapes.
- Every flag write triggers the brain via the shared evaluate helper (the golden wiring rule).
- Real-time task delivery via Firestore listeners. Never polling.
- All documents keyed by residentId. June stores evidence (quotes, counts, durations), never labels.

---

## Appendix — parked for Phase 2

Observer protocol (acoustic anomaly detection), code-blue tap beacon with live audio, nurse drop-in calls, OS-level kiosk pinning, live weather canvas, smart volume normalization, day-one onboarding flow.

Parked with reasons: scope (each is days of work), consent (always-on acoustic analysis deepens the hardest open question), and boundary (beacon features invite the fall-alarm assumption the product explicitly refuses).
